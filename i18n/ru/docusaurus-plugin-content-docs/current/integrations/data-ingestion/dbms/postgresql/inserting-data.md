---
slug: /integrations/postgresql/inserting-data
title: 'Как вставить данные из PostgreSQL'
keywords: ['postgres', 'postgresql', 'вставки']
description: 'Страница, описывающая, как вставить данные из PostgreSQL с помощью ClickPipes, PeerDB или табличной функции PostgreSQL'
---

Мы рекомендуем прочитать [это руководство](/guides/inserting-data), чтобы узнать о лучших практиках вставки данных в ClickHouse для оптимизации производительности вставок.

Для пакетной загрузки данных из PostgreSQL пользователи могут использовать:

- используя [ClickPipes](/integrations/clickpipes/postgres), управляемый сервис интеграции для ClickHouse Cloud - сейчас в публичном бета-тестировании. Пожалуйста, [зарегистрируйтесь здесь](https://clickpipes.peerdb.io/)
- `PeerDB от ClickHouse`, инструмент ETL, специально разработанный для репликации базы данных PostgreSQL как на собственных серверах ClickHouse, так и в ClickHouse Cloud.
    - PeerDB теперь доступен нативно в ClickHouse Cloud - Ультрабыстрая репликация PostgreSQL в ClickHouse CDC с нашим [новым соединителем ClickPipe](/integrations/clickpipes/postgres) - сейчас в публичном бета-тестировании. Пожалуйста, [зарегистрируйтесь здесь](https://clickhouse.com/cloud/clickpipes/postgres-cdc-connector)
- [Табличная функция PostgreSQL](/sql-reference/table-functions/postgresql) для прямого чтения данных. Это обычно подходит для пакетной репликации на основе известного контрольного знака, например, временной метки, или если это одноразовая миграция. Этот подход может масштабироваться до десятков миллионов строк. Пользователям, желающим мигрировать более крупные наборы данных, следует рассмотреть возможность выполнения нескольких запросов, каждый из которых обрабатывает часть данных. Стадии могут использоваться для каждого блока перед перемещением его партиций в конечную таблицу. Это позволяет повторно попытаться выполнить неудавшиеся запросы. Для получения дополнительной информации об этой стратегии пакетной загрузки смотрите здесь.
- Данные могут быть экспортированы из Postgres в формате CSV. Эти данные затем могут быть вставлены в ClickHouse либо из локальных файлов, либо через объектное хранилище с помощью табличных функций.
