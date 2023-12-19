[META]:

```
title: Шпоргалка по MySQL
revision: 2
tags: rdbms, mysql
meta-desc: Шпаргалка по MySQL и MariaDB. Базы данных и пользователи. Работа с дампами. Работа с JSON. Отладка запросов.
```

---

Шпаргалка по типовым, а также редко используемым операциям в MySQL||MariaDB, позволяющая не заглядывать в объемный справочник, по тривиальным вопросам.

---

1. [Базы данных и пользователи](#1-базы-данных-и-пользователи)
    1. [Создать базу](#11-создать-базу)
    2. [Удалить базу](#12-удалить-базу)
    3. [Создать пользователя](#13-создать-пользователя)
    4. [Выдать пользователю полные права на базу](#14-выдать-пользователю-полные-права-на-базу)
    5. [Удалить пользователя](#15-удалить-пользователя)
    6. [Атомарно создать базу и пользователя для нее](#16-атомарно-создать-базу-и-пользователя-для-нее)
    7. [Вывести список баз, доступных пользователю](#17-вывести-список-баз-доступных-пользователю)
    8. [Вывести список таблиц конкретной базы данных](#18-вывести-список-таблиц-конкретной-базы-данных)
    9. [Вычислить размер баз, доступных пользователю](#19-вычислить-размер-баз-доступных-пользователю)
    10. [Вычислить размер таблиц в базе данных](#110-вычислить-размер-таблиц-в-базе-данных)
    11. [Узнать кодировку базы данных](#111-узнать-кодировку-базы-данных)
    12. [Сохранить результат выборки в файл](#112-сохранить-результат-выборки-в-файл)
2. [Работа с дампами](#2-работа-с-дампами)
    1. [Создать дамп](#21-создать-дамп)
    2. [Создать сжатый дамп](#22-создать-сжатый-дамп)
    3. [Создать дамп схемы](#23-создать-дамп-схемы)
    4. [Создать дамп только заданных таблиц](#24-создать-дамп-только-заданных-таблиц)
    5. [Создать дамп с указанием текущей даты-времени](#25-создать-дамп-с-указанием-текущей-даты-времени)
    6. [Создать оптимизированный дамп](#26-создать-оптимизированный-дамп)
    7. [Загрузить дамп](#27-загрузить-дамп)
    8. [Загрузить сжатый дамп](#28-загрузить-сжатый-дамп)
3. [Работа с JSON](#3-работа-с-json)
    1. [Перечень функций](#31-перечень-функций)
    2. [Создание таблицы с JSON-столбцом](#32-создание-таблицы-с-json-столбцом)
    3. [Вставка](#33-вставка)
    4. [Выборка](#34-выборка)
    5. [Обновление/замена](#35-обновлениезамена)
    6. [Удаление](#36-удаление)
    7. [Сравнение и сортировка](#37-сравнение-и-сортировка)
4. [Отладка запросов](#4-отладка-запросов)
    1. [Логирование запросов](#41-логирование-запросов)
    2. [Анализ запросов](#42-анализ-запросов)
    3. [Профилирование запросов](#43-профилирование-запросов)

# 1. Базы данных и пользователи

## 1.1 Создать базу

    :::sql
    CREATE DATABASE `<dbname>` CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

## 1.2 Удалить базу

    :::sql
    DROP DATABASE `<dbname>`;

## 1.3 Создать пользователя

    :::sql
    CREATE USER '<username>'@'<localhost>' IDENTIFIED BY '<password>';

## 1.4 Выдать пользователю полные права на базу

    :::sql
    GRANT ALL PRIVILEGES ON <dbname>.* TO '<username>'@'<localhost>';
    FLUSH PRIVILEGES;

## 1.5 Удалить пользователя

    :::sql
    DROP USER '<username>'@'<localhost>';

> `DROP USER` не закрывает ранее открытые сессии пользователя. Однако попытка открыть новую сесию, от ранее удаленного пользователя, завершится ошибкой авторизации. Это штатное поведение, которое нужно иметь в виду.

## 1.6 Атомарно создать базу и пользователя для нее

    :::sql
    BEGIN;
    CREATE DATABASE `<dbname>` CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
    CREATE USER '<username>'@'<localhost>' IDENTIFIED BY '<password>';
    GRANT ALL PRIVILEGES ON <dbname>.* TO '<username>'@'<localhost>';
    FLUSH PRIVILEGES;
    COMMIT;

## 1.7 Вывести список баз, доступных пользователю

    :::shell-session
    $ mysqlshow -h HOST -u USER -pPASSWORD

## 1.8 Вывести список таблиц конкретной базы данных

    :::shell-session
    $ mysqlshow -h HOST -u USER -pPASSWORD DATABASE

## 1.9 Вычислить размер баз, доступных пользователю

    :::sql
      SELECT t.Database,
             t.Bytes,
             ROUND(t.Bytes / 1024, 2) AS 'KiB',
             ROUND(t.Bytes / 1024 / 1024, 2) AS 'MiB',
             ROUND(t.Bytes / 1024 / 1024 / 1024, 2) AS 'GiB'
        FROM (
                 SELECT table_schema AS 'Database',
                        SUM(data_length + index_length) AS 'Bytes'
                   FROM information_schema.tables
                  WHERE table_schema NOT IN ('information_schema', 'performance_schema')
               GROUP BY table_schema
             ) AS t
    ORDER BY t.Bytes DESC;

> В запросе исключаются служебные базы (`information_schema`, `performance_schema`). Если данные по ним нужны — уберите соответствующее WHERE-выражение.

## 1.10 Вычислить размер таблиц в базе данных

    :::sql
    SELECT table_name AS 'Table',
           ROUND(((data_length + index_length) / 1024 / 1024), 2) AS 'MiB'
      FROM information_schema.TABLES
     WHERE table_schema = '<dbname>';

## 1.11 Узнать кодировку базы данных

    :::sql
    SELECT DEFAULT_CHARACTER_SET_NAME AS 'Charset',
           DEFAULT_COLLATION_NAME AS 'Collation'
      FROM information_schema.SCHEMATA
     WHERE SCHEMA_NAME = '<dbname>';


## 1.12 Сохранить результат выборки в файл

    :::sql
          SELECT ...
            FROM <table>
           WHERE ...
    INTO OUTFILE '/tmp/my_select_result.txt';

Или в виде csv:

    :::sql
                  SELECT ...
                    FROM <table>
                   WHERE ...
            INTO OUTFILE '/tmp/my_select_result.txt'
    FIELDS TERMINATED BY ','
             ENCLOSED BY '"'
     LINES TERMINATED BY '\n';

Или csv с названиями полей:

    :::sql
                  SELECT 'field1', 'field2', 'field3'
               UNION ALL
                  SELECT field1, field2, field3
                    FROM <table>
                   WHERE ...
            INTO OUTFILE '/tmp/my_select_result.txt'
    FIELDS TERMINATED BY ','
             ENCLOSED BY '"'
     LINES TERMINATED BY '\n';

Если в запросе на выборку используется сортировка, то названия заголовков станут частью данных, которые будут упорядоченны (т.е. названия csv-столбцов могут оказаться не первой строкой). Чтобы такого не произошло, можно воспользоваться подзапросом:

    :::sql
          SELECT *
            FROM (
                      SELECT 'field1', 'field2', 'field3'
                   UNION ALL
                      SELECT field1, field2, field3
                        FROM <table>
                    ORDER BY field2 ASC
                 ) AS sub
    INTO OUTFILE '/tmp/my_select_result.txt';

# 2 Работа с дампами

> Создание текстового дампа (*.sql) — способ, приемлемо работающий для небольших баз (до 10-20 гигабайт, в зависимости ОТ). Для создания дампа баз бОльшего объема, следует использовать специализированные инструменты, работающие с бинарными данными.

## 2.1 Создать дамп

    :::shell-session
    $ mysqldump -h HOST -u USER -pPASSWORD DATABASE > DATABASE.sql

## 2.2 Создать сжатый дамп

    :::shell-session
    $ mysqldump -h HOST -u USER -pPASSWORD DATABASE | gzip -9 > DATABASE.sql.gz

## 2.3 Создать дамп схемы

Создается дамп только со структурой таблиц указанной базы, БЕЗ самих данных.

    :::shell-session
    $ mysqldump --no-data -h HOST -u USER -pPASSWORD DATABASE > DATABASE_schema.sql

## 2.4 Создать дамп только заданных таблиц

    :::shell-session
    $ mysqldump -h HOST -u USER -pPASSWORD DATABASE TABLE1 TABLE2 > DATABASE_table1_table2.sql

## 2.5 Создать дамп с указанием текущей даты-времени

    :::shell-session
    $ mysqldump -h HOST -u USER -pPASSWORD DATABASE > `date +DATABASE_%Y%m%d_%H%M%S.sql`

## 2.6 Создать оптимизированный дамп

    :::shell-session
    $ mysqldump --skip-lock-tables \
                --single-transaction \
                -h HOST -u USER -pPASSWORD DATABASE \
                | gzip -9 > `date +DATABASE_%Y%m%d_%H%M%S.sql.gz`

где:

`--skip-lock-tables` — не блокирует таблицы на чтение/запись во время создания дампа;

`--single-transaction` — создает дамп в рамках единой транзакции (состояние БД "замораживается" на момент создания дампа), для обеспечения гарантированной консистентности данных.

## 2.7 Загрузить дамп

    :::shell-session
    $ mysql -h HOST -u USER -pPASSWORD DATABASE < DATABASE.sql

### 2.8 Загрузить сжатый дамп

    :::shell-session
    $ gunzip < DATABASE.sql.gz | mysql -h HOST -u USER -pPASSWORD DATABASE

или

    :::shell-session
    $ zcat DATABASE.sql.gz | mysql -h HOST -u USER -pPASSWORD DATABASE

# 3. Работа с JSON

## 3.1 Перечень функций

### ->

Получить значение из JSON-объекта (или столбца) по указанному path (эквивалент функции `JSON_EXTRACT()`):

    :::sql
    SELECT column->'$.key' ...
    -- "value"

### ->>

Получить значение по заданному JSON-path и выполнить деквотирование (удаление кавычек). Эквивалентно комбинации функций `JSON_UNQUOTE(JSON_EXTRACT())`:

    :::sql
    SELECT column->>"$.key" ...
    -- value

### JSON_ARRAY([val[, val] ...])

Создать JSON-массив из переданных в качестве аргументов значений:

    :::sql
    SELECT JSON_ARRAY(1, 2, 3, "4", 5.0);
    -- [1, 2, 3, "4", 5.0]

### JSON_ARRAY_APPEND(json_doc, path, val[, path, val] ...)

Добавить элемент в конец JSON-массива:

    :::sql
    SET @j = '[1, 2, 3]';

    SELECT JSON_ARRAY_APPEND(@j, '$', 4);
    -- [1, 2, 3, 4]

    SELECT JSON_ARRAY_APPEND(@j, '$[2]', 4);
    -- [1, 2, [3, 4]]

### JSON_ARRAY_INSERT(json_doc, path, val[, path, val] ...)

Вставить элемент в указанную позицию JSON-массива (при этом все последующие элементы автоматически сдвигаются вправо):

    :::sql
    SET @j = '[1, 2, 3]';
    SELECT JSON_ARRAY_INSERT(@j, '$[2]', 4);
    -- [1, 2, 4, 3]

### JSON_CONTAINS(target, candidate[, path])

Проверить, содержится ли в документе искомое ЗНАЧЕНИЕ по указанному JSON-path (возвращает 1 в случае успеха и 0 в противном случае):

    :::sql
    SET @j = '{"a": 100, "b": 500}';

    SELECT JSON_CONTAINS(@j, '100', '$.a');
    -- 1

    SELECT JSON_CONTAINS(@j, '500', '$.a');
    -- 0

### JSON_CONTAINS_PATH(json_doc, one_or_all, path[, path] ...)

Проверить, содержит ли документ искомый "path" внутри себя. Данная проверка имеет два режима работы: "one" — проверяет есть ли в документе хотя бы один из искомых path, режим 'all' — проверяет наличие ВСЕХ искомых path's в документе.

    :::sql
    SET @j = '{"a": 100, "b": 500}';

    SELECT JSON_CONTAINS_PATH(@j, 'one', '$.a');
    -- 1

    SELECT JSON_CONTAINS_PATH(@j, 'all', '$.a', '$.b');
    -- 1

    SELECT JSON_CONTAINS_PATH(@j, 'all', '$.a', '$.b', '$.c');
    -- 0

### JSON_DEPTH(json_doc)

Возвращает коэффициент максимальной глубины JSON-документа:

    :::sql
    SET @j = '{}';
    SELECT JSON_DEPTH(@j);
    -- 1

    SET @j = '{"a": {"b": [{"c": {"d": [1]}}]}}';
    SELECT JSON_DEPTH(@j);
    -- 7

### JSON_EXTRACT(json_doc, path[, path] ...)

Получить значение из JSON-объекта (или столбца) по указанному path (эквивалент функции `->`):

    :::sql
    SET @j = '[1, [2, 3]]';

    SELECT JSON_EXTRACT(@j, '$[0]');
    -- 1

    SELECT JSON_EXTRACT(@j, '$[1][*]');
    -- [2, 3]

    SELECT JSON_EXTRACT(@j, '$[1][1]');
    -- 3

    -- Для выборки данных из JSON-столбца
    SELECT JSON_EXTRACT(json_column, '$.foo') ...
    -- "bar"

### JSON_INSERT(json_doc, path, val[, path, val] ...)

Вставить данные в документ по указанному path:

    :::sql
    SET @j = '{"a": 1, "b": [2, 3]}';
    SELECT JSON_INSERT(@j, '$.a', 10, '$.c', '[true, false]');
    -- {"a": 1, "b": [2, 3], "c": "[true, false]"}

> Пары path, val для существующего в документе path — не изменяется. Для изменения/обновления данных следует использовать функции `JSON_REPLACE()` или `JSON_SET()`.

### JSON_KEYS(json_doc[, path])

Получить массив ключей документа, относительно указанного path:

    :::sql
    SET @j = '{"a": 1, "b": 2, "c": {"d": 4}}';

    SELECT JSON_KEYS(@j);
    -- ["a", "b", "c"]

    SELECT JSON_KEYS(@j, '$.c');
    -- ["d"]

### JSON_LENGTH(json_doc[, path])

Получить длину (количество пар "ключ-значение") JSON-документа, относительно указанного path:

    :::sql
    SET @j = '{"a": 1, "b": 2, "c": {"d": 4}}';

    SELECT JSON_LENGTH(@j);
    -- 3

    SELECT JSON_LENGTH(@j, '$.c');
    -- ["d"]

### JSON_MERGE_PATCH(json_doc, json_doc[, json_doc] ...)

Объединить документы JSON, заменив значения повторяющихся ключей:

    :::sql
    SELECT JSON_MERGE_PATCH('[1, 2]', '[true, false]');
    -- [true, false]

    SELECT JSON_MERGE_PATCH('{"name": "x"}', '{"id": 47}');
    -- {"id": 47, "name": "x"}

    SELECT JSON_MERGE_PATCH('{ "a": 1, "b":2 }', '{ "a": 3, "c":4 }');
    -- {"a": 3, "b": 2, "c": 4}

    SELECT JSON_MERGE_PATCH(
      '{ "a": 1, "b":2 }',
      '{ "a": 3, "c":4 }',
      '{ "a": 5, "d":6 }'
    );
    -- {"a": 5, "b": 2, "c": 4, "d": 6}

### JSON_MERGE_PRESERVE(json_doc, json_doc[, json_doc] ...)

Объединить документы JSON, сохранив значения повторяющихся ключей:

    :::sql
    SELECT JSON_MERGE_PRESERVE('[1, 2]', '[true, false]');
    -- [1, 2, true, false]

    SELECT JSON_MERGE_PRESERVE('{"name": "x"}', '{"id": 47}');
    -- {"id": 47, "name": "x"}

    SELECT JSON_MERGE_PRESERVE('{ "a": 1, "b": 2 }', '{ "a": 3, "c": 4 }');
    -- {"a": [1, 3], "b": 2, "c": 4}

    SELECT JSON_MERGE_PATCH(
      '{ "a": 1, "b":2 }',
      '{ "a": 3, "c":4 }',
      '{ "a": 5, "d":6 }'
    );
    -- {"a": [1, 3, 5], "b": 2, "c": 4, "d": 6}

### JSON_OBJECT([key, val[, key, val] ...])

Создать JSON-объект:

    :::sql
    SELECT JSON_OBJECT('id', 123, 'foo', 'bar', 'spam', JSON_OBJECT('egg', 1));
    -- {"id": 123, "foo": "bar", "spam": {"egg": 1}}

### JSON_PRETTY(json_val)

Распечатать JSON-документ в человекочитаемом формате:

    :::sql
    SELECT JSON_PRETTY("[1,3,5]");
    /*
        [
          1,
          3,
          5
        ]
    */

    SELECT JSON_PRETTY('{"a":"10","b":"15","x":"25"}');
    /*
        {
          "a": "10",
          "b": "15",
          "x": "25"
        }
    */

### JSON_QUOTE(string)

Квотировать (обрамить кавычками, экранируя вложенные) JSON-документ:

    :::sql
    SELECT JSON_QUOTE('null'), JSON_QUOTE('"null"');
    -- "null" | "\"null\""

    SELECT JSON_QUOTE('[1, 2, 3]');
    -- "[1, 2, 3]"

### JSON_REMOVE(json_doc, path[, path] ...)

Удалить данные из JSON-документа по указанному path:

    :::sql
    SET @j = '["a", ["b", "c"], "d"]';
    SELECT JSON_REMOVE(@j, '$[1]');
    -- ["a", "d"]

    SET @j = '{"a": 1, "b": 2}';
    SELECT JSON_REMOVE(@j, '$.a');
    -- {"b": 2}

### JSON_REPLACE(json_doc, path, val[, path, val] ...)

Заменить существующие значения в JSON-документе:

> JSON_REPLACE() — работает только с существующими значениями, новые (ранее отсутствующие) значения не будут добавлены.

    :::sql
    SET @j = '{ "a": 1, "b": [2, 3]}';
    SELECT JSON_REPLACE(@j, '$.a', 10, '$.b[1]', 5, '$.c', '[true, false]');
    -- {"a": 10, "b": [2, 5]}

### JSON_SEARCH(json_doc, one_or_all, search_str[, escape_char[, path] ...])

Возвращает path в документе JSON по искомой строке.

> Возвращает NULL, если: любой из аргументов json\_doc, search\_str или path равен NULL; в документе нет указанного path; или search\_str не найден.

    :::sql
    SET @j = '["abc", [{"k": "10"}, "def"], {"x":"abc"}, {"y":"bcd"}]';
    SELECT JSON_SEARCH(@j, 'one', 'abc');
    -- "$[0]"

    SELECT JSON_SEARCH(@j, 'all', 'abc');
    -- ["$[0]", "$[2].x"]

    JSON_SEARCH(@j, 'all', 'ghi')
    -- NULL

    SELECT JSON_SEARCH(@j, 'all', '10');
    -- "$[1][0].k"

    SELECT JSON_SEARCH(@j, 'all', '10', NULL, '$');
    -- "$[1][0].k"

    SELECT JSON_SEARCH(@j, 'all', '10', NULL, '$[*]');
    -- "$[1][0].k"

    SELECT JSON_SEARCH(@j, 'all', '10', NULL, '$**.k');
    -- "$[1][0].k"

    SELECT JSON_SEARCH(@j, 'all', '10', NULL, '$[*][0].k');
    -- "$[1][0].k"

    SELECT JSON_SEARCH(@j, 'all', '10', NULL, '$[1]');
    -- "$[1][0].k"

    SELECT JSON_SEARCH(@j, 'all', '10', NULL, '$[1][0]');
    -- "$[1][0].k"

    SELECT JSON_SEARCH(@j, 'all', 'abc', NULL, '$[2]');
    -- "$[2].x"

    SELECT JSON_SEARCH(@j, 'all', '%a%');
    -- ["$[0]", "$[2].x"]

    SELECT JSON_SEARCH(@j, 'all', '%b%');
    -- ["$[0]", "$[2].x", "$[3].y"]

    SELECT JSON_SEARCH(@j, 'all', '%b%', '', '$[1]');
    -- NULL

    SELECT JSON_SEARCH(@j, 'all', '%b%', '', '$[3]');
    -- "$[3].y"

### JSON_SET(json_doc, path, val[, path, val] ...)

Вставить данные в JSON-документ:

    :::sql
    SET @j = '{ "a": 1, "b": [2, 3]}';

    SELECT JSON_SET(@j, '$.a', 10, '$.c', '[true, false]');
    -- {"a": 10, "b": [2, 3], "c": "[true, false]"}

    SELECT JSON_INSERT(@j, '$.a', 10, '$.c', '[true, false]');
    -- {"a": 1, "b": [2, 3], "c": "[true, false]"}

    SELECT JSON_REPLACE(@j, '$.a', 10, '$.c', '[true, false]');
    -- {"a": 10, "b": [2, 3]}

>  Функции `JSON_SET()`, `JSON_INSERT()`, и `JSON_REPLACE()` связаны:
> `JSON_SET()` — заменяет существующие и добавляет несуществущие значения
> `JSON_INSERT()` — вставляет новые значения, без изменения существующих
> `JSON_REPLACE()` — заменяет только существующие значения (новые не добавляет)

    :::sql
    SELECT JSON_SET(@j, '$.a', 10, '$.c', '[true, false]');
    -- {"a": 10, "b": [2, 3], "c": "[true, false]"}

    SELECT JSON_INSERT(@j, '$.a', 10, '$.c', '[true, false]');
    -- {"a": 1, "b": [2, 3], "c": "[true, false]"}

    SELECT JSON_REPLACE(@j, '$.a', 10, '$.c', '[true, false]');
    -- {"a": 10, "b": [2, 3]}

### JSON_STORAGE_SIZE(json_val)

Функция возвращает объем (в байтах) данных, необходимых для хранения документа JSON:

    :::sql
    CREATE TABLE jtable (jcol JSON);
    INSERT INTO jtable VALUES ('{"a": 1000, "b": "wxyz", "c": "[1, 3, 5, 7]"}');
    SELECT JSON_STORAGE_SIZE(jcol) FROM jtable;
    -- 47

### JSON_TYPE(json_val)

Возвращает строку (в кодировке `utf8mb4`), указывающую на тип значения JSON (объект, массив или скаляр):

    :::sql
    SET @j = '{"a": [10, true], "b": null}';
    SELECT JSON_TYPE(@j);
    -- OBJECT

    SELECT JSON_TYPE(JSON_EXTRACT(@j, '$.a'));
    -- ARRAY

    SELECT JSON_TYPE(JSON_EXTRACT(@j, '$.a[0]'));
    -- INTEGER

    SELECT JSON_TYPE(JSON_EXTRACT(@j, '$.a[1]'));
    -- BOOLEAN

    SELECT JSON_TYPE(JSON_EXTRACT(@j, '$.b'));
    -- NULL

### JSON_UNQUOTE(json_val)

Функция деквотирует JSON-значение и возвращает результат в виде utf8mb4-строки:

    ::sql
    SET @j = '"abc"';
    SELECT @j, JSON_UNQUOTE(@j);
    -- abc

### JSON_VALID(val)

Проверяет, является ли переданное значение валидным JSON, возвращает 1 или 0 соответственно:

    :::sql
    SELECT JSON_VALID('{"a": 1}');
    -- 1

    SELECT JSON_VALID('hello'), JSON_VALID('"hello"');
    -- 0 | 1

## 3.2 Создание таблицы с JSON-столбцом

    :::sql
    CREATE TABLE <table> (<json_column> JSON);

## 3.3 Вставка

> Для представления json-объекта и json-массива, можно использовать как строковое представление, обрамленное в одинарные кавычки, так и специализированные функции — `JSON_OBJECT` и `JSON_ARRAY` соответственно.

    :::sql
    INSERT INTO <table> (<json_column>)
         VALUES '{"foo": "bar", "pi": 3.14, "second_level": {"key": [1, 2, 3]}}';

или:

    :::sql
    INSERT INTO <table> (<json_column>)
         VALUES JSON_OBJECT(
                    "foo", "bar",
                    "pi", 3.14,
                    "second_level", JSON_OBJECT(
                        "key", JSON_ARRAY(1, 2, 3)
                    )
                );

## 3.4 Выборка

> Для доступа к значению json-обекта по ключу можно использовать как короткий синтаксис — `->`, так и функцию `JSON_EXTRACT`.

    :::sql
    SELECT <field>
      FROM <table>
     WHERE <json_column>->"$.<first_level_key>" = <some_value>;

или:

    :::sql
    SELECT <field>
      FROM <table>
     WHERE JSON_EXTRACT(<json_column>, "$.<first_level_key>") = <some_value>;

## 3.5 Обновление/замена

    :::sql
    UPDATE <table>
       SET <json_column> = JSON_SET(<json_column>, "$.<first_level_key>", <new_value>)
     WHERE <statement>;

## 3.6 Удаление

    :::sql
    UPDATE <table>
       SET <json_column> = JSON_REMOVE(<json_column>, "$.<first_level_key>")
     WHERE <statement>;

## 3.7 Сравнение и сортировка

> JSON-значения могут сравниваться с помощью операторов: =, <, <=, >, >=, <>, != и <=>.
> Однако следующие операторы и функции НЕ работают с JSON-значениями: BETWEEN, IN(), GREATEST(), LEAST(), ORDER BY. Поэтому, JSON-значения в данных операторах и функциях должны быть предварительно конвретированы в примитивы с помощью функции CAST().

    :::sql
    ...
    ORDER BY CAST(JSON_EXTRACT(jdoc, '$.id') AS UNSIGNED)

# 4. Отладка запросов

Нижеописанные средства анализа, вкупе со средствами логирования, позволяют достаточно быстро и эффективно находить проблемные места в коде клиентского приложения, при работе с СУБД.

## 4.1 Логирование запросов

Включенное логирования дает информацию о том, какие именно запросы, и в каком количестве, генерирует тот или иной клиент (клиентское приложение). Особенно актуально, если в приложении для работы с БД используется ORM или иная, схожая по концепции, абстракция.

Просмотр лога — первый этап при отладке запросов. Им же может все и закончиться, так как откровенная дичь, как правило, заметна сразу. По виду бажного запроса, можно быстро найти соответствующий ему код в приложении, и исправить его.

Включение логирования запросов в таблицу:

    :::sql
    SET GLOBAL general_log = 'ON';
    SET GLOBAL log_output = 'TABLE';

Или в файл:

    :::sql
    SET GLOBAL log_output = 'FILE';

> Назание файла для логирования запросов можно задать либо в конфигурации сервера (my.conf: `general_log_file = <filepath>`), либо интерактивно через `SET GLOBAL general_log_file='<filepath>'`. По умолчанию логи записываются в файл: `/var/log/mysql/mysql.log`.

Способ включения *табличных* логов является предпочтительным, так как не требует доступа к файловой системе, и позволяет делать выборки целевых запросов с фильтрацией по нескольким критериям, без более сложного парсинга файлов. Также не стоит забывать о возможности перенаправить результат любого запроса в файл.

Структура таблицы `mysql.general_log`:

| Field         | Type                | Null | Key | Default               | Extra                           |
|---------------|---------------------|------|-----|-----------------------|---------------------------------|
| event\_time   | timestamp(6)        | NO   |     | current\_timestamp(6) | on update current\_timestamp(6) |
| user\_host    | mediumtext          | NO   |     | NULL                  |                                 |
| thread\_id    | bigint(21) unsigned | NO   |     | NULL                  |                                 |
| server\_id    | int(10) unsigned    | NO   |     | NULL                  |                                 |
| command\_type | varchar(64)         | NO   |     | NULL                  |                                 |
| argument      | mediumtext          | NO   |     | NULL                  |                                 |

Выборка запросов к конкретной базе:

    :::sql
    SELECT * FROM mysql.general_log;
    SELECT * FROM mysql.general_log WHERE argument LIKE 'databasename%';

Выборка запросов в интересующем временном интервале:

    :::sql
    SELECT * FROM mysql.general_log
     WHERE event_time > (NOW() - INTERVAL 10 SECOND)
       AND argument LIKE 'databasename%'
     ...

Если логов слишком много, и запрос на выборку начинает тормозить, — можно периодически чистить таблицу:

    :::sql
    TRUNCATE TABLE mysql.general_log;

По завершению работ, логи следует отключить:

    :::sql
    SET GLOBAL general_log = 'OFF';

## 4.2 Анализ запросов

Чтобы проанализировать запрос на семантику, и узнать как этот запрос интерпретирует внутри себя СУБД, достаточно просмотреть план запроса, применив оператор EXPLAIN.

EXPLAIN работает с запросами: SELECT, UPDATE, INSERT, DELETE, REPLACE и предоставляет информацию о типе запроса, используемых в запросе таблицах, используемых индексах, ключах, количестве прочитанных и отфильтрованных строк и т.д.

Кратко про колонки в выводе команды EXPLAIN:

| Колонка        | Краткое описание                                                                                                            |
|----------------|-----------------------------------------------------------------------------------------------------------------------------|
| id             | Идентификатор запроса.                                                                                                      |
| select\_type   | Тип запроса SELECT: SIMPLE, SUBQUERY, DERIVED, UNION, UNION RESULT, MATERIALIZED.                                           |
| table          | Название таблицы.                                                                                                           |
| partitions     | Название используемой партиции (если не используется - null).                                                               |
| type           | ALL, index, range, index\_subquery, unique\_subquery, index\_merge, ref_or_null, fulltext, ref, eq_ref, const, system, NULL |
| possible\_keys | Показывает, какие индексы можно, потенциально, использовать для запроса.                                                    |
| key            | Указывает на ключ (индекс), который оптимизатор MySQL решил использовать фактически.                                        |
| key\_len       | Показывает длину выбранного ключа (индекса) в байтах.                                                                       |
| ref            | Показывает, какие столбцы или константы сравниваются с указанным в key индексом.                                            |
| rows           | Показывает примерное количество строк, которые будут прочитаны.                                                             |
| filtered       | Показывает, какую долю (т.е. %) от общего количества просмотренных строк вернет движок СУБД.                                |
| Extra          | Различная доп. инфа: предупреждения, подсказки и проч.                                                                      |

Описание всех возможных значений в выхлопе EXPLAIN выходит за рамки данной шпаргалки. Тот случай, когда лучше прочитать офф. документацию к вашей версии СУБД.

Здесь, можно ограничиться лишь общим подходом:

*хорошо*, когда в выводе EXPLAIN используются ожидаемые таблицы, индексы, кеш, количество прочитанных и отфильтрованных строк — не зашкаливающее, а в значении Extra нет негативных сообщений или предупреждений;

*плохо*, когда в выводе EXPLAIN большое количество rows, запрос строится не от той таблицы (актуально при неправильно сконструированном JOIN или подзапросах), не используются индексы, кеш, перебираются все записи, а в Extra содержатся ворнинги.

## 4.3 Профилирование запросов

Профилирование позволяет оценить сложность и время выполнения запроса. Для этого применяется выражение `EXPLAIN ANALYZE`. Применяться оно к тем же операторам, что и EXPLAIN.

Схематично, вывод команды можно представить в виде дерева, каждый узел которого оформлен строкой, вида:

`-> [операция] (используемые ключи) (оценка стоимости) (фактические время выполнения операции)`

Пример вывода EXPLAIN ANALIZE:

    :::
    -> Nested loop inner join (cost=1.05 rows=1) (actual time=0.152..0.152 rows=0 loops=1)
      -> Nested loop inner join (cost=0.70 rows=1) (actual time=0.123..0.123 rows=0 loops=1)
        -> Index scan on Drivers using Drivers_car_id_index (cost=0.35 rows=1) (actual time=0.094..0.094 rows=0 loops=1)
        -> Index lookup on Orders using Orders_Drivers_id_fk (driver_id=Drivers.id) (cost=0.35 rows=1) (never executed)
      -> Single-row index lookup on Clients using PRIMARY (id=Orders.client_id) (cost=0.35 rows=1) (never executed)

Каждая вложенная строка означает дополнительное действие/цикл.

`cost` — некая внутренняя оценка (стоимость) того, насколько "дорого" для СУБД выполнять то или иное действие (больше — хуже).

`actual time` — фактическое время получения первой строки и фактическое время получения всех строк, в формате `actual time={время получения первой строки}..{время получения всех строк}`.

`rows` — фактическое количество прочитанных/обработанных строк.

`loops` — количество циклов, которые будут выполнены для соединения с внешней таблицей (выше по дереву). Если не потребовалось ни одной итерации цикла, будет отображено `never executed`.
