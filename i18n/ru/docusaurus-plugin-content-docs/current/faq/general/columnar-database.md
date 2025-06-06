---
slug: /faq/general/columnar-database
title: 'Что такое столбцовая база данных?'
toc_hidden: true
toc_priority: 101
description: 'Эта страница описывает, что такое столбцовая база данных'
---

import Image from '@theme/IdealImage';
import RowOriented from '@site/static/images/row-oriented.gif';
import ColumnOriented from '@site/static/images/column-oriented.gif';


# Что такое столбцовая база данных? {#what-is-a-columnar-database}

Столбцовая база данных хранит данные каждой колонки независимо. Это позволяет считывать данные с диска только для тех колонок, которые используются в любом данном запросе. Стоимость заключается в том, что операции, затрагивающие целые строки, становятся пропорционально более дорогими. Синонимом столбцовой базы данных является система управления базами данных, ориентированная на столбцы. ClickHouse является典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典典лексий. 

Ключевыми преимуществами столбцовой базы данных являются:

- Запросы, которые используют только несколько колонок из многих.
- Агрегирующие запросы к большим объемам данных.
- Сжатие данных по колонкам.

Вот иллюстрация различия между традиционными системами, ориентированными на строки, и столбцовыми базами данных при построении отчетов:

**Традиционная система, ориентированная на строки**
<Image img={RowOriented} alt="Традиционная база данных, ориентированная на строки" size="md" border />

**Столбцовая**
<Image img={ColumnOriented} alt="Столбцовая база данных" size="md" border />

Столбцовая база данных является предпочтительным выбором для аналитических приложений, поскольку она позволяет иметь много колонок в таблице на всякий случай, но не оплачивать стоимость неиспользуемых колонок во времени выполнения запросов (традиционная OLTP база данных считывает все данные во время выполнения запросов, поскольку данные хранятся в строках, а не в колонках). Базы данных, ориентированные на столбцы, предназначены для обработки больших данных и хранилищ данных, они часто масштабируются с использованием распределенных кластеров недорогого оборудования для увеличения пропускной способности. ClickHouse делает это, комбинируя [распределенные](../../engines/table-engines/special/distributed.md) и [реплицированные](../../engines/table-engines/mergetree-family/replication.md) таблицы.

Если вы хотите углубиться в историю столбцовых баз данных, в чем их отличие от систем, ориентированных на строки, и случаи использования столбцовых баз данных, смотрите [руководство по столбцовым базам данных](https://clickhouse.com/engineering-resources/what-is-columnar-database).
