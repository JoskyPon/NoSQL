# Отчет по лабораторной работе №2  
**Изучение Apache HBase: основы работы с распределенной NoSQL базой данных**

---

##  Задание 1

### Информационный поиск

1. **Изучение операций удаления в оболочке HBase**  
   - `delete` - удаление отдельного значения столбца в строке:
     ```bash
     hbase> delete 'table_name', 'row_key', 'column_family:column_qualifier'
     ```
   - `deleteall` - удаление всей строки:
     ```bash
     hbase> deleteall 'table_name', 'row_key'
     ```

2. **Документация по API HBase**  
   Добавлена в закладки: [HBase API Documentation v1.2.1](https://hbase.apache.org/1.2/apidocs/)

### Практические задачи

1. **Создание функции `put_many()`**  
   ```ruby
   def put_many(table_name, row, column_values)
     import 'org.apache.hadoop.hbase.client.HTable'
     import 'org.apache.hadoop.hbase.client.Put'
     
     def jbytes(*args)
       args.map { |arg| arg.to_s.to_java_bytes }
     end
     
     table = HTable.new(@hbase.configuration, table_name)
     p = Put.new(*jbytes(row))
     
     column_values.each do |column, value|
       family, qualifier = column.split(':', 2)
       p.add(*jbytes(family, qualifier, value))
     end
     
     table.put(p)
   end

    # Отчет по лабораторной работе №2  

##  Задание 2

### Информационный поиск

1. **Сжатие в HBase: плюсы и минусы**  
   **Преимущества:**
   - Значительная экономия дискового пространства (до 70-80% для текстовых данных)
   - Ускорение операций чтения за счет уменьшения объема передаваемых данных
   - Снижение сетевой нагрузки в распределенных кластерах
   - Улучшение производительности при сканировании больших объемов данных

   **Недостатки:**
   - Дополнительная нагрузка на процессор при сжатии и распаковке данных
   - Возможное замедление операций записи
   - Некоторые алгоритмы (LZO) требуют дополнительной установки и настройки
   - Усложнение отладки и мониторинга

2. **Фильтры Блума в HBase**  
   - **Принцип работы:** вероятностная структура данных для быстрой проверки отсутствия элемента
   - **Типы в HBase:**
     - `ROW` - проверка существования строки
     - `ROWCOL` - проверка существования конкретного столбца в строке
   - **Преимущества:** предотвращение дорогостоящих операций чтения с диска
   - **Особенности:** возможны ложноположительные срабатывания, но никогда ложные отрицания

3. **Параметры семейства столбцов, связанные со сжатием**  
   ```bash
   hbase> alter 'table_name', {NAME => 'cf', 
     COMPRESSION => 'GZ',           # Алгоритм сжатия
     BLOOMFILTER => 'ROWCOL',       # Тип фильтра Блума
     BLOCKSIZE => '65536',          # Размер блоков
     VERSIONS => '3',               # Количество версий
     COMPRESSION_COMPACT => 'GZ'    # Сжатие для компактных файлов
   }

### Практические задачи

**Создание таблицы foods для хранения данных о продуктах питания**
Команда создания таблицы:

bash
hbase> create 'foods', {
  NAME => 'facts',
  VERSIONS => 1,
  BLOOMFILTER => 'ROW',
  COMPRESSION => 'GZ',
  BLOCKSIZE => '65536'
}
**Обоснование выбора параметров:**

  - Ключ строки: Food_Code - уникальный идентификатор продукта

  - BLOOMFILTER => 'ROW': ускорение проверки существования продуктов

  - COMPRESSION => 'GZ': эффективное сжатие текстовых данных

  - BLOCKSIZE => '65536': оптимальный размер для баланса производительности

  - VERSIONS => 1: хранение только последней версии данных

**2.2 Скрипт импорта данных о продуктах питания**
Полный код скрипта:

ruby
require 'time'
import 'org.apache.hadoop.hbase.client.HTable'
import 'org.apache.hadoop.hbase.client.Put'
import 'javax.xml.stream.XMLStreamConstants'

def jbytes(*args)
  args.map { |arg| arg.to_s.to_java_bytes }
end

factory = javax.xml.stream.XMLInputFactory.newInstance
reader = factory.createXMLStreamReader(java.lang.System.in)

document = nil
buffer = nil
count = 0

table = HTable.new(@hbase.configuration, 'foods')
table.setAutoFlush(false)

while reader.has_next
  type = reader.next
  
  if type == XMLStreamConstants::START_ELEMENT
    case reader.local_name
    when 'Food_Display_Row' 
      document = {}
    when /Food_Code|Display_Name|Portion_Display_Name|Calories|Portion_Amount/ 
      buffer = []
    end
    
  elsif type == XMLStreamConstants::CHARACTERS
    buffer << reader.text unless buffer.nil?
    
  elsif type == XMLStreamConstants::END_ELEMENT
    case reader.local_name
    when /Food_Code|Display_Name|Portion_Display_Name|Calories|Portion_Amount/
      document[reader.local_name] = buffer.join
      
    when 'Food_Display_Row'
      # Используем Food_Code как ключ строки
      key = document['Food_Code'].to_java_bytes
      p = Put.new(key)
      
      # Добавляем все доступные поля в семейство столбцов facts
      p.add(*jbytes("facts", "Display_Name", document['Display_Name'])) if document['Display_Name']
      p.add(*jbytes("facts", "Portion_Display_Name", document['Portion_Display_Name'])) if document['Portion_Display_Name']
      p.add(*jbytes("facts", "Calories", document['Calories'])) if document['Calories']
      p.add(*jbytes("facts", "Portion_Amount", document['Portion_Amount'])) if document['Portion_Amount']
      
      table.put(p)
      count += 1
      
      # Периодическая фиксация изменений для оптимизации производительности
      table.flushCommits() if count % 50 == 0
      
      # Вывод прогресса
      puts "#{count} продуктов импортировано (#{document['Display_Name']})" if count % 200 == 0
    end
  end
end

# Финальная фиксация оставшихся изменений
table.flushCommits()
puts "Импорт завершен. Всего обработано: #{count} записей"
exit
2.3 Запрос данных о продуктах через оболочку HBase
Запуск импорта данных:

bash
$ cat Food_Display_Table.xml | ${HBASE_HOME}/bin/hbase shell import_foods.rb
Примеры запросов к данным:

bash
# Получение данных о конкретном продукте
hbase> get 'foods', '1001'

# Получение только названия продукта
hbase> get 'foods', '2005', 'facts:Display_Name'

# Просмотр первых 5 записей
hbase> scan 'foods', {LIMIT => 5}

# Просмотр названий и калорийности первых 10 продуктов
hbase> scan 'foods', {COLUMNS => ['facts:Display_Name', 'facts:Calories'], LIMIT => 10}

# Подсчет общего количества записей
hbase> count 'foods', INTERVAL => 1000