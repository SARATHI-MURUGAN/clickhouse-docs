---
sidebar_label: 'HTTP Sink Connector для Confluent Platform'
sidebar_position: 3
slug: /integrations/kafka/cloud/confluent/http
description: 'Использование HTTP Connector Sink с Kafka Connect и ClickHouse'
title: 'Confluent HTTP Sink Connector'
---

import ConnectionDetails from '@site/docs/_snippets/_gather_your_details_http.mdx';
import Image from '@theme/IdealImage';
import createHttpSink from '@site/static/images/integrations/data-ingestion/kafka/confluent/create_http_sink.png';
import httpAuth from '@site/static/images/integrations/data-ingestion/kafka/confluent/http_auth.png';
import httpAdvanced from '@site/static/images/integrations/data-ingestion/kafka/confluent/http_advanced.png';
import createMessageInTopic from '@site/static/images/integrations/data-ingestion/kafka/confluent/create_message_in_topic.png';



# Confluent HTTP Sink Connector
HTTP Sink Connector является независимым от типа данных и, следовательно, не требует схемы Kafka, а также поддерживает специфичные для ClickHouse типы данных, такие как Maps и Arrays. Эта дополнительная гибкость требует немного больше усилий при настройке.

Ниже мы опишем простую установку, получая сообщения из одной темы Kafka и вставляя строки в таблицу ClickHouse.

:::note
  HTTP Connector распространяется под [лицензией Confluent Enterprise](https://docs.confluent.io/kafka-connect-http/current/overview.html#license).
:::

### Быстрые шаги {#quick-start-steps}

#### 1. Соберите свои данные для подключения {#1-gather-your-connection-details}
<ConnectionDetails />


#### 2. Запустите Kafka Connect и HTTP Sink Connector {#2-run-kafka-connect-and-the-http-sink-connector}

У вас есть два варианта:

* **Самоуправление:** Скачайте пакет Confluent и установите его локально. Следуйте инструкциям по установке коннектора, описанным [здесь](https://docs.confluent.io/kafka-connect-http/current/overview.html).
Если вы используете метод установки confluent-hub, ваши локальные файлы конфигурации будут обновлены.

* **Confluent Cloud:** Полностью управляемая версия HTTP Sink доступна для тех, кто использует Confluent Cloud для хостинга Kafka. Это требует, чтобы ваша среда ClickHouse была доступна из Confluent Cloud.

:::note
  Следующие примеры используют Confluent Cloud.
:::

#### 3. Создайте целевую таблицу в ClickHouse {#3-create-destination-table-in-clickhouse}

Перед тестированием подключения давайте начнем с создания тестовой таблицы в ClickHouse Cloud, эта таблица будет получать данные из Kafka:

```sql
CREATE TABLE default.my_table
(
    `side` String,
    `quantity` Int32,
    `symbol` String,
    `price` Int32,
    `account` String,
    `userid` String
)
ORDER BY tuple()
```

#### 4. Настройте HTTP Sink {#4-configure-http-sink}
Создайте тему Kafka и экземпляр HTTP Sink Connector:
<Image img={createHttpSink} size="sm" alt="Интерфейс Confluent Cloud, показывающий, как создать HTTP Sink connector" border/>

<br />

Настройте HTTP Sink Connector:
* Укажите имя темы, которую вы создали
* Аутентификация
    * `HTTP Url` - URL ClickHouse Cloud с указанным запросом `INSERT` `<protocol>://<clickhouse_host>:<clickhouse_port>?query=INSERT%20INTO%20<database>.<table>%20FORMAT%20JSONEachRow`. **Примечание**: запрос должен быть закодирован.
    * `Endpoint Authentication type` - BASIC
    * `Auth username` - имя пользователя ClickHouse
    * `Auth password` - пароль ClickHouse

:::note
  Этот HTTP Url подвержен ошибкам. Убедитесь, что кодирование точное, чтобы избежать проблем.
:::

<Image img={httpAuth} size="lg" alt="Интерфейс Confluent Cloud, показывающий настройки аутентификации для HTTP Sink connector" border/>
<br/>

* Конфигурация
    * `Input Kafka record value format` зависит от ваших исходных данных, но в большинстве случаев это JSON или Avro. Мы предполагаем `JSON` в следующих настройках.
    * В разделе `advanced configurations`:
        * `HTTP Request Method` - Установите на POST
        * `Request Body Format` - json
        * `Batch batch size` - Согласно рекомендациям ClickHouse, установите это на **не менее 1000**.
        * `Batch json as array` - true
        * `Retry on HTTP codes` - 400-500, но адаптируйте по необходимости, например, это может измениться, если у вас есть HTTP-прокси перед ClickHouse.
        * `Maximum Reties` - значение по умолчанию (10) подходит, но можете настроить для более надежных повторов.

<Image img={httpAdvanced} size="sm" alt="Интерфейс Confluent Cloud, показывающий параметры расширенной конфигурации для HTTP Sink connector" border/>

#### 5. Тестирование подключения {#5-testing-the-connectivity}
Создайте сообщение в теме, настроенной вашим HTTP Sink
<Image img={createMessageInTopic} size="md" alt="Интерфейс Confluent Cloud, показывающий, как создать тестовое сообщение в теме Kafka" border/>

<br/>

и проверьте, что созданное сообщение было записано в ваш экземпляр ClickHouse.

### Устранение неполадок {#troubleshooting}
#### HTTP Sink не объединяет сообщения {#http-sink-doesnt-batch-messages}

Согласно [документации Sink](https://docs.confluent.io/kafka-connectors/http/current/overview.html#http-sink-connector-for-cp):
> HTTP Sink connector не объединяет запросы для сообщений, содержащих различные значения заголовков Kafka.

1. Убедитесь, что ваши записи Kafka имеют одинаковый ключ.
2. Когда вы добавляете параметры к URL API HTTP, каждая запись может привести к уникальному URL. По этой причине слияние отключено, когда используются дополнительные параметры URL.

#### 400 Bad Request {#400-bad-request}
##### CANNOT_PARSE_QUOTED_STRING {#cannot_parse_quoted_string}
Если HTTP Sink не удается выполнить вставку JSON-объекта в колонку `String` с сообщением:

```response
Code: 26. DB::ParsingException: Cannot parse JSON string: expected opening quote: (while reading the value of key key_name): While executing JSONEachRowRowInputFormat: (at row 1). (CANNOT_PARSE_QUOTED_STRING)
```

Установите настройку `input_format_json_read_objects_as_strings=1` в URL в виде закодированной строки `SETTINGS%20input_format_json_read_objects_as_strings%3D1`

### Загрузите набор данных GitHub (необязательно) {#load-the-github-dataset-optional}

Обратите внимание, что этот пример сохраняет массивные поля набора данных Github. Мы предполагаем, что у вас есть пустая тема github в примерах и вы используете [kcat](https://github.com/edenhill/kcat) для вставки сообщений в Kafka.

##### 1. Подготовьте конфигурацию {#1-prepare-configuration}

Следуйте [этим инструкциям](https://docs.confluent.io/cloud/current/cp-component/connect-cloud-config.html#set-up-a-local-connect-worker-with-cp-install) для настройки Connect в зависимости от вашего типа установки, учитывая различия между автономным и распределенным кластером. Если вы используете Confluent Cloud, актуальна распределенная настройка.

Самый важный параметр - это `http.api.url`. [HTTP интерфейс](../../../../interfaces/http.md) для ClickHouse требует, чтобы вы закодировали оператор INSERT в качестве параметра в URL. Это должно включать формат (`JSONEachRow` в данном случае) и целевую базу данных. Формат должен соответствовать данным Kafka, которые будут преобразованы в строку в HTTP полезной нагрузке. Эти параметры должны быть URL закодированы. Пример этого формата для набора данных Github (предполагая, что вы запускаете ClickHouse локально) показан ниже:

```response
<protocol>://<clickhouse_host>:<clickhouse_port>?query=INSERT%20INTO%20<database>.<table>%20FORMAT%20JSONEachRow

http://localhost:8123?query=INSERT%20INTO%20default.github%20FORMAT%20JSONEachRow
```

Следующие дополнительные параметры актуальны для использования HTTP Sink с ClickHouse. Полный список параметров можно найти [здесь](https://docs.confluent.io/kafka-connect-http/current/connector_config.html):

* `request.method` - Установите на **POST**
* `retry.on.status.codes` - Установите на 400-500, чтобы повторить при любых ошибках. Уточните в зависимости от ожидаемых ошибок в данных.
* `request.body.format` - В большинстве случаев это будет JSON.
* `auth.type` - Установите на BASIC, если у вас есть безопасность с ClickHouse. Другие совместимые с ClickHouse механизмы аутентификации в данный момент не поддерживаются.
* `ssl.enabled` - установите на true, если используете SSL.
* `connection.user` - имя пользователя для ClickHouse.
* `connection.password` - пароль для ClickHouse.
* `batch.max.size` - Количество строк, отправляемых в одном пакете. Убедитесь, что это значение установлено на достаточно большое число. Согласно рекомендациям ClickHouse, значением 1000 следует считать минимум.
* `tasks.max` - HTTP Sink connector поддерживает выполнение одной или нескольких задач. Это может использоваться для увеличения производительности. Вместе с размером пакета это ваши основные способы улучшения производительности.
* `key.converter` - установите в зависимости от типов ваших ключей.
* `value.converter` - установите в зависимости от типа данных в вашей теме. Эти данные не требуют схемы. Формат здесь должен соответствовать формату, указанному в параметре `http.api.url`. Проще всего использовать JSON и конвертер org.apache.kafka.connect.json.JsonConverter. Также возможно рассматривать значение как строку, используя конвертер org.apache.kafka.connect.storage.StringConverter, хотя это потребует от пользователя извлечения значения в операторах вставки с использованием функций. Также поддерживается [формат Avro](../../../../interfaces/formats.md#data-format-avro) в ClickHouse, если используется конвертер io.confluent.connect.avro.AvroConverter.

Полный список настроек, включая то, как настроить прокси, повторы и расширенные настройки SSL, можно найти [здесь](https://docs.confluent.io/kafka-connect-http/current/connector_config.html).

Примеры файлов конфигурации для образца данных Github можно найти [здесь](https://github.com/ClickHouse/clickhouse-docs/tree/main/docs/integrations/data-ingestion/kafka/code/connectors/http_sink), предполагая, что Connect запускается в автономном режиме, а Kafka хостится в Confluent Cloud.

##### 2. Создайте таблицу ClickHouse {#2-create-the-clickhouse-table}

Убедитесь, что таблица создана. Пример для минимального набора данных github, используя стандартный MergeTree, показан ниже.

```sql
CREATE TABLE github
(
    file_time DateTime,
    event_type Enum('CommitCommentEvent' = 1, 'CreateEvent' = 2, 'DeleteEvent' = 3, 'ForkEvent' = 4,'GollumEvent' = 5, 'IssueCommentEvent' = 6, 'IssuesEvent' = 7, 'MemberEvent' = 8, 'PublicEvent' = 9, 'PullRequestEvent' = 10, 'PullRequestReviewCommentEvent' = 11, 'PushEvent' = 12, 'ReleaseEvent' = 13, 'SponsorshipEvent' = 14, 'WatchEvent' = 15, 'GistEvent' = 16, 'FollowEvent' = 17, 'DownloadEvent' = 18, 'PullRequestReviewEvent' = 19, 'ForkApplyEvent' = 20, 'Event' = 21, 'TeamAddEvent' = 22),
    actor_login LowCardinality(String),
    repo_name LowCardinality(String),
    created_at DateTime,
    updated_at DateTime,
    action Enum('none' = 0, 'created' = 1, 'added' = 2, 'edited' = 3, 'deleted' = 4, 'opened' = 5, 'closed' = 6, 'reopened' = 7, 'assigned' = 8, 'unassigned' = 9, 'labeled' = 10, 'unlabeled' = 11, 'review_requested' = 12, 'review_request_removed' = 13, 'synchronize' = 14, 'started' = 15, 'published' = 16, 'update' = 17, 'create' = 18, 'fork' = 19, 'merged' = 20),
    comment_id UInt64,
    path String,
    ref LowCardinality(String),
    ref_type Enum('none' = 0, 'branch' = 1, 'tag' = 2, 'repository' = 3, 'unknown' = 4),
    creator_user_login LowCardinality(String),
    number UInt32,
    title String,
    labels Array(LowCardinality(String)),
    state Enum('none' = 0, 'open' = 1, 'closed' = 2),
    assignee LowCardinality(String),
    assignees Array(LowCardinality(String)),
    closed_at DateTime,
    merged_at DateTime,
    merge_commit_sha String,
    requested_reviewers Array(LowCardinality(String)),
    merged_by LowCardinality(String),
    review_comments UInt32,
    member_login LowCardinality(String)
) ENGINE = MergeTree ORDER BY (event_type, repo_name, created_at)

```

##### 3. Добавьте данные в Kafka {#3-add-data-to-kafka}

Вставьте сообщения в Kafka. Ниже мы используем [kcat](https://github.com/edenhill/kcat) для вставки 10k сообщений.

```bash
head -n 10000 github_all_columns.ndjson | kcat -b <host>:<port> -X security.protocol=sasl_ssl -X sasl.mechanisms=PLAIN -X sasl.username=<username>  -X sasl.password=<password> -t github
```

Простое чтение из целевой таблицы "Github" должно подтвердить вставку данных.

```sql
SELECT count() FROM default.github;

| count\(\) |
| :--- |
| 10000 |

```
