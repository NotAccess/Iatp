# Основы обработки данных с помощью R и Dplyr


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

### Шаг 1: Получение данных

1.  **Установка пакета nycflights13**

    ``` terminal
    install.packages('nycflights13')
    ```

2.  **Загрузка библиотек**

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
    library(nycflights13)
    ```

        Warning: пакет 'nycflights13' был собран под R версии 4.4.2

3.  **Проверка полученных данных**

    ``` r
    knitr::kable(head(flights)[,1:5])
    ```

    <table>
    <thead>
    <tr class="header">
    <th style="text-align: right;">year</th>
    <th style="text-align: right;">month</th>
    <th style="text-align: right;">day</th>
    <th style="text-align: right;">dep_time</th>
    <th style="text-align: right;">sched_dep_time</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td style="text-align: right;">2013</td>
    <td style="text-align: right;">1</td>
    <td style="text-align: right;">1</td>
    <td style="text-align: right;">517</td>
    <td style="text-align: right;">515</td>
    </tr>
    <tr class="even">
    <td style="text-align: right;">2013</td>
    <td style="text-align: right;">1</td>
    <td style="text-align: right;">1</td>
    <td style="text-align: right;">533</td>
    <td style="text-align: right;">529</td>
    </tr>
    <tr class="odd">
    <td style="text-align: right;">2013</td>
    <td style="text-align: right;">1</td>
    <td style="text-align: right;">1</td>
    <td style="text-align: right;">542</td>
    <td style="text-align: right;">540</td>
    </tr>
    <tr class="even">
    <td style="text-align: right;">2013</td>
    <td style="text-align: right;">1</td>
    <td style="text-align: right;">1</td>
    <td style="text-align: right;">544</td>
    <td style="text-align: right;">545</td>
    </tr>
    <tr class="odd">
    <td style="text-align: right;">2013</td>
    <td style="text-align: right;">1</td>
    <td style="text-align: right;">1</td>
    <td style="text-align: right;">554</td>
    <td style="text-align: right;">600</td>
    </tr>
    <tr class="even">
    <td style="text-align: right;">2013</td>
    <td style="text-align: right;">1</td>
    <td style="text-align: right;">1</td>
    <td style="text-align: right;">554</td>
    <td style="text-align: right;">558</td>
    </tr>
    </tbody>
    </table>

### Шаг 2: Ответы на вопросы

1.  **Сколько встроенных в пакет nycflights13 датафреймов?**

    ``` r
    ls("package:nycflights13") %>% length()
    ```

        [1] 5

2.  **Сколько строк в каждом датафрейме?**

    ``` r
    for (df in ls("package:nycflights13")) {
      cat(df, "->", nrow(get(df)), "\n")
    }
    ```

        airlines -> 16 
        airports -> 1458 
        flights -> 336776 
        planes -> 3322 
        weather -> 26115 

3.  **Сколько столбцов в каждом датафрейме?**

    ``` r
    for (df in ls("package:nycflights13")) {
      cat(df, "->", ncol(get(df)), "\n")
    }
    ```

        airlines -> 2 
        airports -> 8 
        flights -> 19 
        planes -> 9 
        weather -> 15 

4.  **Как просмотреть примерный вид датафрейма?**

    ``` r
    airlines %>% glimpse()
    ```

        Rows: 16
        Columns: 2
        $ carrier <chr> "9E", "AA", "AS", "B6", "DL", "EV", "F9", "FL", "HA", "MQ", "O…
        $ name    <chr> "Endeavor Air Inc.", "American Airlines Inc.", "Alaska Airline…

5.  **Сколько компаний-перевозчиков (carrier) учитывают эти наборы
    данных (представлено в наборах данных)?**

    ``` r
    airlines %>%
      select(carrier) %>%
      unique() %>%
      filter(!is.na(carrier)) %>%
      nrow()
    ```

        [1] 16

6.  **Сколько рейсов принял аэропорт John F Kennedy Intl в мае?**

    ``` r
    faa_JFKI <- airports %>%
      filter(name == "John F Kennedy Intl") %>%
      select(faa) %>%
      c()

    flights %>%
      filter(month == 5 & origin == faa_JFKI) %>%
      nrow()
    ```

        [1] 9397

7.  **Какой самый северный аэропорт?**

    ``` r
    airports %>%
      arrange(desc(lat)) %>%
      head(1) %>%
      select(name) %>%
      knitr::kable()
    ```

    <table>
    <thead>
    <tr class="header">
    <th style="text-align: left;">name</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td style="text-align: left;">Dillant Hopkins Airport</td>
    </tr>
    </tbody>
    </table>

8.  **Какой аэропорт самый высокогорный (находится выше всех над уровнем
    моря)?**

    ``` r
    airports %>%
      arrange(desc(alt)) %>%
      head(1) %>%
      select(name) %>%
      knitr::kable()
    ```

    <table>
    <thead>
    <tr class="header">
    <th style="text-align: left;">name</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td style="text-align: left;">Telluride</td>
    </tr>
    </tbody>
    </table>

9.  **Какие бортовые номера у самых старых самолетов?**

    ``` r
    planes %>%
      arrange(year) %>%
      head(1) %>%
      select(tailnum) %>%
      knitr::kable()
    ```

    <table>
    <thead>
    <tr class="header">
    <th style="text-align: left;">tailnum</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td style="text-align: left;">N381AA</td>
    </tr>
    </tbody>
    </table>

10. **Какая средняя температура воздуха была в сентябре в аэропорту John
    F Kennedy Intl (в градусах Цельсия)?**

    ``` r
    weather %>%
      filter(month == 9 & origin == faa_JFKI) %>%
      mutate(temp_C = (5/9) * (temp - 32)) %>%
      summarise(avg_temp_C = mean(temp_C, na.rm = TRUE)) %>%
      knitr::kable()
    ```

    <table>
    <thead>
    <tr class="header">
    <th style="text-align: right;">avg_temp_C</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td style="text-align: right;">19.38764</td>
    </tr>
    </tbody>
    </table>

11. **Самолеты какой авиакомпании совершили больше всего вылетов в
    июне?**

    ``` r
    carrier_June <- flights %>%
      filter(month == 6) %>%
      count(carrier) %>%
      arrange(desc(n)) %>%
      head(1) %>%
      select(carrier) %>%
      c()

    airlines %>%
      filter(carrier == carrier_June) %>%
      select(name) %>%
      knitr::kable()
    ```

    <table>
    <thead>
    <tr class="header">
    <th style="text-align: left;">name</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td style="text-align: left;">United Air Lines Inc.</td>
    </tr>
    </tbody>
    </table>

12. **Самолеты какой авиакомпании задерживались чаще других в 2013
    году?**

    ``` r
    carrier_delay <- flights %>%
      filter(arr_delay > 0) %>%
      count(carrier) %>%
      arrange(desc(n)) %>%
      head(1) %>%
      select(carrier) %>%
      c()

    airlines %>%
      filter(carrier == carrier_delay) %>%
      select(name) %>%
      knitr::kable()
    ```

    <table>
    <thead>
    <tr class="header">
    <th style="text-align: left;">name</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td style="text-align: left;">ExpressJet Airlines Inc.</td>
    </tr>
    </tbody>
    </table>

## ️Оценка результата

Были использованы знания функций
`select(), filter(), mutate(), arrange(), group_by()` для решения
практических задач.

## ️Вывод

В результате выполнения работы были:

-   развиты практические навыки использования языка программирования R
    для обработки данных
-   закреплены знания базовых типов данных языка R
-   развиты практические навыки использования функций обработки данных
    пакета `dplyr` – функции
    `select(), filter(), mutate(), arrange(), group_by()`
