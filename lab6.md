# Отчет по выполнению лабораторной работы №6
## Работа с Amazon DynamoDB

### Задание 1: Информационный поиск

#### 1.1. Формула расчета количества разделов в DynamoDB

Согласно документации AWS, количество разделов для таблицы DynamoDB рассчитывается по формуле:
  - Number of partitions = (Total RCU / 3000) + (Total WCU / 1000)


Где:
- **RCU** (Read Capacity Units) - единицы пропускной способности чтения
- **WCU** (Write Capacity Units) - единицы пропускной способности записи

Ограничения на один раздел:
- Максимум 3000 RCU
- Максимум 1000 WCU

#### 1.2. Функция потоков DynamoDB

DynamoDB Streams - это функция, которая захватывает изменения элементов в таблице DynamoDB в хронологическом порядке. Каждое изменение представляется как потоковая запись.

**Возможности использования:**
- Реализация триггеров для реагирования на изменения данных
- Репликация данных между таблицами
- Анализ данных в реальном времени
- Аудитинг изменений данных

#### 1.3. Ограничения DynamoDB

Основные ограничения DynamoDB:
- **Размер элемента**: 400 KB
- **Имена таблиц**: 3-255 символов, только буквы, цифры, подчеркивания, точки и дефисы
- **Атрибуты на элемент**: Нет явного ограничения, но ограничен общим размером 400 KB
- **Размер ключа раздела**: 1-2048 байт
- **Размер ключа сортировки**: 1-1024 байт

### Задание 2: Практические задачи

#### 2.1. Расчет количества разделов

**Дано:**
- Объем данных: 100 ГБ
- RCU: 2000
- WCU: 3000

**Расчет:**

Number of partitions = (2000 / 3000) + (3000 / 1000)
= 0.67 + 3
= 3.67 ≈ 4 раздела


**Ответ:** Для заданных параметров потребуется 4 раздела.

#### 2.2. Хранение танцев в DynamoDB

Для хранения информации о танцах можно использовать следующую структуру:

```json
{
  "DanceId": {"S": "waltz-001"},
  "DanceName": {"S": "Viennese Waltz"},
  "OriginCountry": {"S": "Austria"},
  "TimeSignature": {"S": "3/4"},
  "Difficulty": {"N": "3"},
  "Styles": {"SS": ["Ballroom", "Classical"]},
  "Characteristics": {
    "M": {
      "Tempo": {"S": "Fast"},
      "Movement": {"S": "Rotating"},
      "Hold": {"S": "Closed"}
    }
  },
  "PopularIn": {"L": [
    {"S": "Austria"},
    {"S": "Germany"},
    {"S": "International"}
  ]}
}

# Создаем элемент для обновления
aws dynamodb put-item \
  --table-name ShoppingCart \
  --item '{
    "ItemName": {"S": "Test Item"},
    "Quantity": {"N": "5"},
    "Price": {"N": "19.99"}
}'

#### 2.3. Условное обновление элемента в таблице ShoppingCart

# Условное обновление - увеличиваем количество только если текущее значение меньше 10
aws dynamodb update-item \
  --table-name ShoppingCart \
  --key '{"ItemName": {"S": "Test Item"}}' \
  --update-expression "SET Quantity = Quantity + :inc" \
  --condition-expression "Quantity < :max" \
  --expression-attribute-values '{
    ":inc": {"N": "1"},
    ":max": {"N": "10"}
  }' \
  --return-values ALL_NEW

```json
{
  "Attributes": {
    "ItemName": {"S": "Test Item"},
    "Quantity": {"N": "6"},
    "Price": {"N": "19.99"}
  }
}
**Проверка с условием, которое не выполняется:**

```bash
# Попытка обновить, когда Quantity уже 6 (больше 5)
aws dynamodb update-item \
  --table-name ShoppingCart \
  --key '{"ItemName": {"S": "Test Item"}}' \
  --update-expression "SET Quantity = Quantity + :inc" \
  --condition-expression "Quantity < :max" \
  --expression-attribute-values '{
    ":inc": {"N": "1"},
    ":max": {"N": "5"}
  }'
**Ожидаемая ошибка:**

```json
{
  "error": "ConditionalCheckFailedException",
  "message": "The conditional request failed"
}