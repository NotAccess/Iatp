# Основы обработки данных с помощью R и Dplyr
KDA

## Цель

1.  Изучить возможности СУБД [DuckDB](https://duckdb.org//) для
    обработки и анализ больших данных
2.  Получить навыки применения DuckDB совместно с языком
    программирования R
3.  Получить навыки анализа `метаинфомации о сетевом трафике`
4.  Получить навыки применения облачных технологий хранения, подготовки
    и анализа данных: [Yandex Object
    Storage](https://yandex.cloud/ru/docs/storage/), [Rstudio
    Server](https://posit.co/products/open-source/rstudio-server/)

## ️Исходные данные

1.  R 4.4.1
2.  RStudio 2024.04.2+764
3.  Yandex Cloud

## ️Общий план выполнения

Используя R и среду разработки RStudio IDE, выполнить задания.

## Содержание ЛР

### Шаг 1: Настройка duckdb

1.  **Установить программный пакет duckdb с помощью:**

``` r
library(duckdb)
```

    Warning: пакет 'duckdb' был собран под R версии 4.4.2

    Загрузка требуемого пакета: DBI

    Warning: пакет 'DBI' был собран под R версии 4.4.2

2.  **Загрузить остальные библиотеки**

``` r
library(DBI)
library(dplyr)
```

    Warning: пакет 'dplyr' был собран под R версии 4.4.2


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

``` r
library(utils)
```

3.  **Загрузим данные в СУБД**

``` r
con <- dbConnect(duckdb::duckdb())
path <- file.path("D:", "C practise", "datasets", "tm_data.pqt")

dbExecute(con, paste0("CREATE TABLE test AS SELECT * FROM read_parquet('", path, "')"))
```

    [1] 105747730

``` r
dbGetQuery(con, "SELECT * from test LIMIT 10")
```

          timestamp           src          dst port bytes
    1  1.578326e+12   13.43.52.51 18.70.112.62   40 57354
    2  1.578326e+12 16.79.101.100  12.48.65.39   92 11895
    3  1.578326e+12 18.43.118.103  14.51.30.86   27   898
    4  1.578326e+12 15.71.108.118 14.50.119.33   57  7496
    5  1.578326e+12  14.33.30.103  15.24.31.23  115 20979
    6  1.578326e+12 18.121.115.31  13.56.39.74   92  8620
    7  1.578326e+12  16.108.75.29  14.34.34.69   65 46033
    8  1.578326e+12 12.46.104.126  16.25.76.33  123  1500
    9  1.578326e+12   12.43.98.93  18.85.31.68   79   979
    10 1.578326e+12  14.32.60.107 12.30.62.113   72  1036

### Шаг 2: Выполнение заданий

1.  **Надите утечку данных из Вашей сети**

Важнейшие документы с результатами нашей исследовательской деятельности
в области создания вакцин скачиваются в виде больших заархивированных
дампов. Один из хостов в нашей сети используется для пересылки этой
информации – он пересылает гораздо больше информации на внешние ресурсы
в Интернете, чем остальные компьютеры нашей сети. Определите его
IP-адрес

``` r
#echo: false
#con <- dbConnect(duckdb::duckdb())
query="SELECT DISTINCT
*
FROM
'test'
where dst like '12.%' or dst like '13.%' or dst like '14.%'

UNION

SELECT DISTINCT
*
FROM
'test'
where src like '12.%' or src like '13.%' or src like '14.%';"
#vnutr_addrs <- dbGetQuery(con, query)

res_query<- "WITH vnutr_addr AS (
  SELECT DISTINCT
  dst
  FROM
  'test'
  where dst like '12.%' or dst like '13.%' or dst like '14.%'
  
  UNION
  
  SELECT DISTINCT
  src
  FROM
  'test'
  where src like '12.%' or src like '13.%' or src like '14.%'
) 

SELECT src, SUM(bytes) FROM test
WHERE src in 
  (
    SELECT DISTINCT
    dst
    FROM
    'test'
    where dst like '12.%' or dst like '13.%' or dst like '14.%'
    
    UNION
    
    SELECT DISTINCT
    src
    FROM
    'test'
    where src like '12.%' or src like '13.%' or src like '14.%'
  )  
and dst not in 
  (
    SELECT DISTINCT
    dst
    FROM
    'test'
    where dst like '12.%' or dst like '13.%' or dst like '14.%'
    
    UNION
    
    SELECT DISTINCT
    src
    FROM
    'test'
    where src like '12.%' or src like '13.%' or src like '14.%'
  ) 
GROUP BY src
ORDER BY SUM(bytes) DESC
LIMIT 1;"

dbGetQuery(con, res_query)
```

               src  sum(bytes)
    1 13.37.84.125 10625497574

2.  **Найдите утечку данных 2**

Другой атакующий установил автоматическую задачу в системном
планировщике cron для экспорта содержимого внутренней wiki системы. Эта
система генерирует большое количество трафика в нерабочие часы, больше
чем остальные хосты. Определите IP этой системы. Известно, что ее IP
адрес отличается от нарушителя из предыдущей задачи.

``` r
query2 = "WITH traffic AS (
    SELECT
        extract(hour FROM epoch_ms(cast(timestamp as bigint))) AS hour,
        src,
        SUM(bytes) AS total_bytes
    FROM
        test
    WHERE
        SUBSTRING(src, 1, POSITION('.' IN src) - 1)::INT BETWEEN 12 AND 14
        and src!='13.37.84.125'
    GROUP BY
        hour, src
),
max_traffic AS (
    SELECT
        hour,
        src,
        total_bytes,
        RANK() OVER (PARTITION BY hour ORDER BY total_bytes DESC) AS rank
    FROM
        traffic
)
SELECT
    src
FROM
    max_traffic
WHERE
    rank = 1
    and hour in (
      SELECT extract(hour FROM epoch_ms(cast(timestamp as bigint))) as hour FROM 'test'
           WHERE (src LIKE '12.%' or src LIKE '13.%' or src LIKE '14.%') and
           NOT (dst LIKE '12.%' or dst LIKE '13.%' or dst LIKE '14.%')
           GROUP BY hour
           ORDER BY STDDEV(bytes) DESC
           LIMIT 1)


"
dbGetQuery(con, query2)
```

              src
    1 12.55.77.96

3.  **Надите утечку данных 3**

Еще один нарушитель собирает содержимое электронной почты и отправляет в
Интернет используя порт, который обычно используется для другого типа
трафика. Атакующий пересылает большое количество информации используя
этот порт, которое нехарактерно для других хостов, использующих этот
номер порта. Определите IP этой системы. Известно, что ее IP адрес
отличается от нарушителей из предыдущих задач.

``` r
query3="
-- Шаг 1: Суммирование трафика по IP и порту
WITH TrafficSummary AS (
    SELECT 
        src AS ip_address,
        port,
        SUM(bytes) AS total_bytes
    FROM 
        test
     WHERE (src LIKE '12.%' or src LIKE '13.%' or src LIKE '14.%') and 
           NOT (dst LIKE '12.%' or dst LIKE '13.%' or dst LIKE '14.%')
    GROUP BY 
        src, port
),

-- Шаг 2: Вычисление среднего трафика по порту
AverageTraffic AS (
    SELECT 
        port,
        AVG(total_bytes) AS avg_bytes
    FROM 
        TrafficSummary
    GROUP BY 
        port
)

-- Шаг 3: Определение отклонений
SELECT 
    ts.ip_address,
    ts.port,
    ts.total_bytes,
    at.avg_bytes,
    ABS(ts.total_bytes - at.avg_bytes) AS deviation
FROM 
    TrafficSummary ts
JOIN 
    AverageTraffic at ON ts.port = at.port
WHERE 
    ABS(ts.total_bytes - at.avg_bytes) > (2 * at.avg_bytes)
ORDER BY 
    ts.port, ts.ip_address;

"
dbGetQuery(con, query3)
```

       ip_address port total_bytes avg_bytes deviation
    1 12.30.96.87  124      356207  20599.12  335607.9

## ️Оценка результата

Провели анализ сетевого трафика с помощью библиотеки Duckdb

## ️Вывод

1.  Установлен пакет in-memory СУБД Duckdb
2.  Выполнены задания
3.  Составлен отчет
