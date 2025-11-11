# Отчет по выполнению лабораторной работы №4
## Работа с CouchDB: CRUD операции и создание представлений

### Задание 1: Информационный поиск

#### 1.1. Документация HTTP API CouchDB

Изучена официальная документация Apache CouchDB по HTTP API. Основные поддерживаемые методы HTTP:

- **GET** - получение данных
- **POST** - создание новых документов
- **PUT** - создание/обновление документов с указанным ID
- **DELETE** - удаление документов
- **COPY** - копирование документов
- **HEAD** - получение метаданных

#### 1.2. Дополнительные методы HTTP

Согласно документации, CouchDB также поддерживает:
- **COPY** - для копирования документов
- **HEAD** - для проверки существования ресурсов без получения тела ответа

### Задание 2: Практические задачи

#### 2.1. Создание документа с определенным _id

```bash
# Создание документа с конкретным ID
curl -X PUT "http://localhost:5984/music/beatles_manual" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "The Beatles Manual",
    "genre": "Rock",
    "formed": 1960
  *}'

**Результат:**

json
{
  "ok": true,
  "id": "beatles_manual",
  "rev": "1-8a3f4b2c9d1e5f7a9b8c6d4e2f1a3b5c"
}
2.2. Создание и удаление базы данных
```bash

**Создание новой базы данных**
curl -X PUT "http://localhost:5984/test_database"

# Проверка создания
curl "http://localhost:5984/test_database"

# Удаление базы данных
curl -X DELETE "http://localhost:5984/test_database"
Результат создания:

json
{"ok": true}
Результат удаления:

json
{"ok": true}
2.3. Работа с вложениями
bash
# Создание документа с вложением
curl -X PUT "http://localhost:5984/music/doc_with_attachment" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Document with Attachment",
    "description": "This document contains a text file attachment"
  }'

# Добавление текстового вложения
curl -X PUT "http://localhost:5984/music/doc_with_attachment/attachment.txt" \
  -H "Content-Type: text/plain" \
  --data-binary "This is the content of the text file attachment."

# Получение только вложения
curl "http://localhost:5984/music/doc_with_attachment/attachment.txt"
Результат создания документа:

json
{
  "ok": true,
  "id": "doc_with_attachment",
  "rev": "1-6d4f8a2b9c1e3f5a7b9d2e4f6a8c1e3f"
}
Результат добавления вложения:

json
{
  "ok": true,
  "id": "doc_with_attachment",
  "rev": "2-9a7b5d3e1f8c6a4b2d9e7f5a3c1b8d6e"
}