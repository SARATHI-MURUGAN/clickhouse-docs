---
slug: /faq/general/olap
title: 'Что такое OLAP?'
toc_hidden: true
toc_priority: 100
description: 'Объяснение, что такое онлайн-аналитическая обработка'
---


# Что такое OLAP? {#what-is-olap}

[OLAP](https://en.wikipedia.org/wiki/Online_analytical_processing) означает онлайн-аналитическую обработку. Это широкий термин, который можно рассматривать с двух точек зрения: технической и бизнес. Но на самом высоком уровне вы можете просто прочитать эти слова в обратном порядке:

Обработка
:   Некоторые исходные данные обрабатываются...

Аналитическая
:   ...чтобы создать некоторые аналитические отчеты и выводы...

Онлайн
:   ...в реальном времени.

## OLAP с точки зрения бизнеса {#olap-from-the-business-perspective}

В последние годы люди в бизнесе начали осознавать ценность данных. Компании, которые принимают свои решения вслепую, чаще всего не успевают за конкурентами. Подход, основанный на данных, успешных компаний заставляет их собирать все данные, которые могут быть удалённо полезны для принятия бизнес-решений, и нуждаются в механизмах для своевременного их анализа. Здесь на помощь приходят системы управления базами данных OLAP.

С точки зрения бизнеса, OLAP позволяет компаниям постоянно планировать, анализировать и отчитываться о операционной деятельности, тем самым максимизируя эффективность, снижая расходы и в конечном итоге завоевывая долю на рынке. Это можно сделать как в рамках внутренней системы, так и путем использования облачных решений от поставщиков SaaS, таких как веб/мобильные аналитические услуги, CRM-сервисы и т.д. OLAP является технологией, лежащей в основе многих приложений BI (бизнес-аналитики).

ClickHouse – это система управления базами данных OLAP, которая довольно часто используется в качестве бэкенда для этих SaaS-решений для анализа специфичных для домена данных. Тем не менее, некоторые компании все еще не хотят делиться своими данными с третьими лицами, и сценарий внутреннего хранилища данных также является жизнеспособным.

## OLAP с технической точки зрения {#olap-from-the-technical-perspective}

Все системы управления базами данных можно классифицировать на две группы: OLAP (онлайн **аналитическая** обработка) и OLTP (онлайн **транзакционная** обработка). Первая группа фокусируется на создании отчетов, каждый из которых основан на больших объемах исторических данных, но делается это не так часто. В то время как вторая группа обычно обрабатывает непрерывный поток транзакций, постоянно изменяя текущее состояние данных.

На практике OLAP и OLTP не являются категориями, это скорее спектр. Большинство реальных систем обычно сосредоточены на одной из них, но предоставляют некоторые решения или обходные пути, если требуется также противоположный вид загрузки. Эта ситуация часто заставляет компании использовать несколько интегрированных систем хранения, что может быть не столь большим делом, но наличие большего количества систем увеличивает затраты на обслуживание. Поэтому тренд последних лет – это HTAP (**гибридная транзакционная/аналитическая обработка**), когда оба вида загрузки обрабатываются столь же эффективно одной системой управления базами данных.

Даже если СУБД начиналась как чистая OLAP или чистая OLTP, им приходится двигаться в сторону HTAP, чтобы не отставать от конкуренции. И ClickHouse не является исключением, изначально он был спроектирован как [OLAP-система с высокой производительностью](../../concepts/why-clickhouse-is-so-fast.md) и до сих пор не имеет полноценной поддержки транзакций, но некоторые функции, такие как согласованное чтение/запись и мутации для обновления/удаления данных, были добавлены.

Основной компромисс между системами OLAP и OLTP остается следующим:

- Для того чтобы эффективно создавать аналитические отчеты, важно иметь возможность читать колонки отдельно, поэтому большинство OLAP баз данных являются [столбцовыми](../../faq/general/columnar-database.md),
- В то время как хранение колонок отдельно увеличивает затраты на операции со строками, такие как добавление или модификация на месте, пропорционально количеству колонок (что может быть огромным, если системы пытаются собрать все детали события на всякий случай). Поэтому большинство OLTP систем хранит данные, организованные по строкам.
