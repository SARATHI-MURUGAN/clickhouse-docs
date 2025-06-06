---
sidebar_label: 'Загрузка данных'
title: 'Загрузка данных из BigQuery в ClickHouse'
slug: /migrations/bigquery/loading-data
description: 'Как загрузить данные из BigQuery в ClickHouse'
keywords: ['migrate', 'migration', 'migrating', 'data', 'etl', 'elt', 'BigQuery']
---

_Этот гид совместим с ClickHouse Cloud и для саморазмещаемого ClickHouse v23.5+._

Этот гид показывает, как мигрировать данные из [BigQuery](https://cloud.google.com/bigquery) в ClickHouse.

Сначала мы экспортируем таблицу в [объектное хранилище Google (GCS)](https://cloud.google.com/storage), а затем импортируем эти данные в [ClickHouse Cloud](https://clickhouse.com/cloud). Эти шаги необходимо повторить для каждой таблицы, которую вы хотите экспортировать из BigQuery в ClickHouse.

## Сколько времени займет экспорт данных в ClickHouse? {#how-long-will-exporting-data-to-clickhouse-take}

Экспорт данных из BigQuery в ClickHouse зависит от размера вашего набора данных. В качестве сравнения, экспорт [публичного набора данных Ethereum размером 4TB](https://cloud.google.com/blog/products/data-analytics/ethereum-bigquery-public-dataset-smart-contract-analytics) из BigQuery в ClickHouse с использованием этого руководства занимает около часа.

| Таблица                                                                                         | Строки        | Экспортировано файлов | Размер данных | Экспорт из BigQuery | Время обработки  | Импорт в ClickHouse |
| ------------------------------------------------------------------------------------------------ | -------------- | --------------------- | ------------- | ------------------- | ---------------- | -------------------- |
| [blocks](https://github.com/ClickHouse/examples/blob/main/ethereum/schemas/blocks.md)           | 16,569,489     | 73                    | 14.53GB       | 23 сек              | 37 мин           | 15.4 сек             |
| [transactions](https://github.com/ClickHouse/examples/blob/main/ethereum/schemas/transactions.md) | 1,864,514,414  | 5169                  | 957GB         | 1 мин 38 сек        | 1 дн 8ч          | 18 мин 5 сек        |
| [traces](https://github.com/ClickHouse/examples/blob/main/ethereum/schemas/traces.md)           | 6,325,819,306  | 17,985                | 2.896TB       | 5 мин 46 сек        | 5 дн 19 ч        | 34 мин 55 сек       |
| [contracts](https://github.com/ClickHouse/examples/blob/main/ethereum/schemas/contracts.md)     | 57,225,837     | 350                   | 45.35GB       | 16 сек              | 1 ч 51 мин       | 39.4 сек             |
| Всего                                                                                           | 8.26 миллиардов | 23,577                | 3.982TB       | 8 мин 3 сек         | > 6 дн 5 ч       | 53 мин 45 сек       |

## 1. Экспорт данных таблицы в GCS {#1-export-table-data-to-gcs}

На этом этапе мы используем [рабочее пространство SQL BigQuery](https://cloud.google.com/bigquery/docs/bigquery-web-ui) для выполнения наших SQL-команд. Ниже мы экспортируем таблицу BigQuery с именем `mytable` в корзину GCS, используя оператор [`EXPORT DATA`](https://cloud.google.com/bigquery/docs/reference/standard-sql/other-statements).

```sql
DECLARE export_path STRING;
DECLARE n INT64;
DECLARE i INT64;
SET i = 0;

-- Рекомендуем установить n, чтобы он соответствовал x миллиардам строк. Итак, 5 миллиардов строк, n = 5
SET n = 100;

WHILE i < n DO
  SET export_path = CONCAT('gs://mybucket/mytable/', i,'-*.parquet');
  EXPORT DATA
    OPTIONS (
      uri = export_path,
      format = 'PARQUET',
      overwrite = true
    )
  AS (
    SELECT * FROM mytable WHERE export_id = i
  );
  SET i = i + 1;
END WHILE;
```

В приведенном выше запросе мы экспортируем нашу таблицу BigQuery в [формат данных Parquet](https://parquet.apache.org/). У нас также есть символ `*` в параметре `uri`. Это гарантирует, что вывод разбивается на несколько файлов с числовыми суффиксами, если экспорт превышает 1 ГБ данных.

Этот подход имеет несколько преимуществ:

- Google позволяет экспортировать до 50TB в день в GCS бесплатно. Пользователи платят только за хранилище GCS.
- Экспорты автоматически создают несколько файлов, ограничивая каждый максимальным объемом данных таблицы в 1 ГБ. Это полезно для ClickHouse, поскольку позволяет параллелить импорты.
- Parquet, как столбцовый формат, представляет собой лучший формат обмена, поскольку он по своей природе сжат и обеспечивается более быстрая скорость экспорта для BigQuery и запросов для ClickHouse.

## 2. Импорт данных в ClickHouse из GCS {#2-importing-data-into-clickhouse-from-gcs}

После завершения экспорта мы можем импортировать эти данные в таблицу ClickHouse. Вы можете использовать [SQL-консоль ClickHouse](/integrations/sql-clients/sql-console) или [`clickhouse-client`](/interfaces/cli) для выполнения приведенных ниже команд.

Сначала необходимо [создать таблицу](/sql-reference/statements/create/table) в ClickHouse:

```sql
-- Если ваша таблица BigQuery содержит колонку типа STRUCT, вы должны включить эту настройку
-- чтобы сопоставить эту колонку с колонкой ClickHouse типа Nested
SET input_format_parquet_import_nested = 1;

CREATE TABLE default.mytable
(
        `timestamp` DateTime64(6),
        `some_text` String
)
ENGINE = MergeTree
ORDER BY (timestamp);
```

После создания таблицы включите настройку `parallel_distributed_insert_select`, если у вас есть несколько реплик ClickHouse в вашем кластере, чтобы ускорить наш экспорт. Если у вас только один узел ClickHouse, вы можете пропустить этот шаг:

```sql
SET parallel_distributed_insert_select = 1;
```

Наконец, мы можем вставить данные из GCS в нашу таблицу ClickHouse, используя команду [`INSERT INTO SELECT`](/sql-reference/statements/insert-into#inserting-the-results-of-select), которая вставляет данные в таблицу на основе результатов запроса `SELECT`.

Чтобы извлечь данные для `INSERT`, мы можем использовать функцию [s3Cluster](/sql-reference/table-functions/s3Cluster) для получения данных из нашей корзины GCS, так как GCS совместим с [Amazon S3](https://aws.amazon.com/s3/). Если у вас только один узел ClickHouse, вы можете использовать [функцию s3](/sql-reference/table-functions/s3) вместо функции `s3Cluster`.

```sql
INSERT INTO mytable
SELECT
    timestamp,
    ifNull(some_text, '') as some_text
FROM s3Cluster(
    'default',
    'https://storage.googleapis.com/mybucket/mytable/*.parquet.gz',
    '<ACCESS_ID>',
    '<SECRET>'
);
```

`ACCESS_ID` и `SECRET`, используемые в приведенном выше запросе, являются вашим [HMAC ключом](https://cloud.google.com/storage/docs/authentication/hmackeys), связанным с вашей корзиной GCS.

:::note Используйте `ifNull`, когда экспортируете колонки с возможными значениями NULL
В приведенном запросе мы используем [`ifNull` функцию](/sql-reference/functions/functions-for-nulls#ifnull) с колонкой `some_text`, чтобы вставить данные в нашу таблицу ClickHouse с значением по умолчанию. Вы также можете сделать свои колонки в ClickHouse [`Nullable`](/sql-reference/data-types/nullable), но это не рекомендуется, поскольку может негативно сказаться на производительности.

В качестве альтернативы, вы можете `SET input_format_null_as_default=1`, и любые отсутствующие или NULL значения будут заменены значениями по умолчанию для соответствующих колонок, если эти значения по умолчанию указаны.
:::

## 3. Тестирование успешного экспорта данных {#3-testing-successful-data-export}

Чтобы проверить, были ли данные правильно вставлены, просто выполните запрос `SELECT` на вашей новой таблице:

```sql
SELECT * FROM mytable limit 10;
```

Чтобы экспортировать больше таблиц BigQuery, просто повторите вышеуказанные шаги для каждой дополнительной таблицы.

## Дополнительные чтения и поддержка {#further-reading-and-support}

В дополнение к этому руководству, мы также рекомендуем прочитать наш блог, который показывает [как использовать ClickHouse для ускорения BigQuery и как обрабатывать инкрементальные импорты](https://clickhouse.com/blog/clickhouse-bigquery-migrating-data-for-realtime-queries).

Если у вас возникли проблемы с передачей данных из BigQuery в ClickHouse, пожалуйста, не стесняйтесь обращаться к нам по адресу support@clickhouse.com.
