# Анализ данных сетевого трафика при помощи библиотеки Arrow
KDA

## Цель

1.  Развить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания базовых типов данных языка R
3.  Развить практические навыки использования функций обработки данных
    пакета `dplyr` – функции
    `select(), filter(), mutate(), arrange(), group_by()`

## ️Исходные данные

1.  R 4.4.1
2.  RStudio 2024.04.2+764

## ️Общий план выполнения

Используя R и среду разработки RStudio IDE, выполнить задания.

## Содержание ЛР

### Шаг 1: Настройка рабочего окружения

1.  **Установить программный пакет arrow с помощью:**

    -   интерфейса RStudio IDE

2.  **Получим данные**

``` r
library(arrow) 
```

    Warning: пакет 'arrow' был собран под R версии 4.4.2


    Присоединяю пакет: 'arrow'

    Следующий объект скрыт от 'package:utils':

        timestamp

``` r
library(dplyr)
```

    Warning: пакет 'dplyr' был собран под R версии 4.4.2


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

``` r
path <- file.path("D:", "C practise", "datasets", "tm_data.pqt")
net_data <- read_parquet(path, as_data_frame = FALSE)
```

### Шаг 2: Выполнение заданий

1.  **Надите утечку данных из Вашей сети**

Важнейшие документы с результатами нашей исследовательской деятельности
в области создания вакцин скачиваются в виде больших заархивированных
дампов. Один из хостов в нашей сети используется для пересылки этой
информации – он пересылает гораздо больше информации на внешние ресурсы
в Интернете, чем остальные компьютеры нашей сети. Определите его
IP-адрес

``` r
net_data %>% 
  filter(grepl("^1[2-4]\\..+", src) & !grepl("^1[2-4]\\..+", dst)) %>%
  group_by(src) %>%
  summarise(traffic = sum(bytes)) %>%
  arrange(desc(traffic)) %>%
  head(1) %>%
  select(src) %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">src</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">13.37.84.125</td>
</tr>
</tbody>
</table>

2.  **Найдите утечку данных 2**

Другой атакующий установил автоматическую задачу в системном
планировщике cron для экспорта содержимого внутренней wiki системы. Эта
система генерирует большое количество трафика в нерабочие часы, больше
чем остальные хосты. Определите IP этой системы. Известно, что ее IP
адрес отличается от нарушителя из предыдущей задачи.

``` r
# Преобразование timestamp в час
net_data <- net_data %>%
  mutate(hour = hour(as_datetime(timestamp / 1000)))
```

``` r
traffic <- net_data %>%
  filter(grepl("^1[2-4]\\..+", src) &
           !grepl("^1[2-4]\\..+", dst) &
           src != '13.37.84.125') %>%
  group_by(hour, src) %>%
  summarise(total_bytes = sum(bytes), .groups = 'drop')

max_traffic <- traffic %>%
  arrange(hour, desc(total_bytes)) %>%
  group_by(hour) %>%
  mutate(rank = row_number()) %>%
  ungroup()
```

    Warning: In row_number(): 
    ℹ Expression not supported in Arrow
    → Pulling data into R

``` r
hour_with_max_stddev <- net_data %>%
  filter(grepl("^1[2-4]\\..+", src) &
           !grepl("^1[2-4]\\..+", dst))%>%
  group_by(hour) %>%
  summarise(stddev_bytes = sd(bytes), .groups = 'drop') %>%
  arrange(desc(stddev_bytes)) %>%
  slice_head(n = 1) %>%
  pull(hour)
```

    Warning: Default behavior of `pull()` on Arrow data is changing. Current behavior of returning an R vector is deprecated, and in a future release, it will return an Arrow `ChunkedArray`. To control this:
    ℹ Specify `as_vector = TRUE` (the current default) or `FALSE` (what it will change to) in `pull()`
    ℹ Or, set `options(arrow.pull_as_vector)` globally
    This warning is displayed once every 8 hours.

``` r
result <- max_traffic %>%
  filter(rank == 1, hour %in% hour_with_max_stddev) %>%
  select(src) %>%
  knitr::kable()
```

``` r
result %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">x</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">|src |</td>
</tr>
<tr class="even">
<td style="text-align: left;">|:———–|</td>
</tr>
<tr class="odd">
<td style="text-align: left;">|12.55.77.96 |</td>
</tr>
</tbody>
</table>

3.  **Надите утечку данных 3**

Еще один нарушитель собирает содержимое электронной почты и отправляет в
Интернет используя порт, который обычно используется для другого типа
трафика. Атакующий пересылает большое количество информации используя
этот порт, которое нехарактерно для других хостов, использующих этот
номер порта. Определите IP этой системы. Известно, что ее IP адрес
отличается от нарушителей из предыдущих задач.

``` r
# Шаг 1: Суммирование трафика по IP и порту
TrafficSummary <- net_data %>%
  filter(grepl("^1[2-4]\\..+", src) & !grepl("^1[2-4]\\..+", dst)) %>%
  group_by(ip_address = src, port) %>%
  summarise(total_bytes = sum(bytes), .groups = 'drop')

# Шаг 2: Вычисление среднего трафика по порту
AverageTraffic <- TrafficSummary %>%
  group_by(port) %>%
  summarise(avg_bytes = mean(total_bytes), .groups = 'drop')

# Шаг 3: Определение отклонений
result <- TrafficSummary %>%
  inner_join(AverageTraffic, by = "port") %>%
  mutate(deviation = abs(total_bytes - avg_bytes)) %>%
  filter(deviation > (2 * avg_bytes)) %>%
  arrange(port, ip_address)

# Вывод результата
print(result)
```

    Table (query)
    ip_address: string
    port: int32
    total_bytes: int64
    avg_bytes: double
    deviation: double (abs_checked(subtract_checked(total_bytes, avg_bytes)))

    * Filter: (abs_checked(subtract_checked(total_bytes, avg_bytes)) > multiply_checked(2, avg_bytes))
    * Sorted by port [asc], ip_address [asc]
    See $.data for the source Arrow object

``` r
result %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">ip_address</th>
<th style="text-align: right;">port</th>
<th style="text-align: right;">total_bytes</th>
<th style="text-align: right;">avg_bytes</th>
<th style="text-align: right;">deviation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">12.30.96.87</td>
<td style="text-align: right;">124</td>
<td style="text-align: right;">356207</td>
<td style="text-align: right;">20599.12</td>
<td style="text-align: right;">335607.9</td>
</tr>
</tbody>
</table>

## ️Оценка результата

Провели анализ сетевого трафика с помощью библиотеки arrow

## ️Вывод

1.  Установлен пакет arrow
2.  Выполнены задания
3.  Составлен отчет
