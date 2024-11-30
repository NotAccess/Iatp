# QUARTO8
KDA

## Задание

Используя язык программирования R, библиотеку duckdb и облачную IDE
Rstudio Server, развернутую в Yandex Cloud, выполнить задания и
составить отчет. Описание полей датасета: timestamp,src,dst,port,bytes
IP адреса внутренней сети начинаются с 12-14 Все остальные IP адреса
относятся к внешним узлам

``` r
#echo: false
library(DBI)
library(duckdb)
library(dplyr)
```


    Attaching package: 'dplyr'

    The following objects are masked from 'package:stats':

        filter, lag

    The following objects are masked from 'package:base':

        intersect, setdiff, setequal, union

``` r
library(utils)
con <- dbConnect(duckdb::duckdb())
query <- "CREATE TABLE test AS SELECT * FROM 'tm_data.parquet';"
dbExecute(con,query)
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

## Задание 1: Надите утечку данных из Вашей сети

Важнейшие документы с результатами нашей исследовательской деятельности
в области создания вакцин скачиваются в виде больших заархивированных
дампов. Один из хостов в нашей сети используется для пересылки этой
информации – он пересылает гораздо больше информации на внешние ресурсы
в Интернете, чем остальные компьютеры нашей сети. Определите его
IP-адрес.

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
ORDER BY SUM(bytes)
LIMIT 1;"

dbGetQuery(con, res_query)
```

              src sum(bytes)
    1 14.42.60.94   10114942

You can add options to executable code like this

``` r
#echo: false
```

The `echo: false` option disables the printing of code (only output is
displayed).

## Задание 3: Надите утечку данных 3

Еще один нарушитель собирает содержимое электронной почты и отправляет в
Интернет используя порт, который обычно используется для другого типа
трафика. Атакующий пересылает большое количество информации используя
этот порт, которое нехарактерно для других хостов, использующих этот
номер порта. Определите IP этой системы. Известно, что ее IP адрес
отличается от нарушителей из предыдущих задач.

``` r
#echo: false
query3="SELECT src, port, SUM(bytes) from test 
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
GROUP BY src, port 
ORDER BY src, port
LIMIT 10"
dbGetQuery(con, query3)
```

                src port sum(bytes)
    1  12.30.105.68   22     694804
    2  12.30.105.68   23     670325
    3  12.30.105.68   25      14364
    4  12.30.105.68   26     685806
    5  12.30.105.68   27     684952
    6  12.30.105.68   29   21971164
    7  12.30.105.68   34     660261
    8  12.30.105.68   37   21500048
    9  12.30.105.68   39   21579679
    10 12.30.105.68   40   21688674

``` r
#echo: false
dbGetQuery(con, query3)
```

                src port sum(bytes)
    1  12.30.105.68   22     694804
    2  12.30.105.68   23     670325
    3  12.30.105.68   25      14364
    4  12.30.105.68   26     685806
    5  12.30.105.68   27     684952
    6  12.30.105.68   29   21971164
    7  12.30.105.68   34     660261
    8  12.30.105.68   37   21500048
    9  12.30.105.68   39   21579679
    10 12.30.105.68   40   21688674
