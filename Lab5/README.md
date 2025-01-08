# Исследование информации о состоянии беспроводных сетей
KDA

## Цель

1.  Получить знания о методах исследования радиоэлектронной обстановки.
2.  Составить представление о механизмах работы Wi-Fi сетей на канальном
    и сетевом уровне модели OSI.
3.  Зекрепить практические навыки использования языка программирования R
    для обработки данных
4.  Закрепить знания основных функций обработки данных экосистемы
    `tidyverse` языка R

## ️Исходные данные

1.  R 4.4.1
2.  RStudio 2024.04.2+764

## ️Общий план выполнения

Используя R и среду разработки RStudio IDE, выполнить задания.

## Содержание ЛР

### Шаг 1: Подготовка данных

1.  **Проверить наличие данных**

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
library(readr)
```

    Warning: пакет 'readr' был собран под R версии 4.4.2

``` r
library(tidyr)
```

    Warning: пакет 'tidyr' был собран под R версии 4.4.2

2.  **Первая часть данных**

``` r
access_points <- read.csv(file="./data/P2_wifi_data.csv", nrows = 167)
```

3.  **Вторая часть**

``` r
clients <- read.csv(file="./data/P2_wifi_data.csv", skip=169)
```

4.  **Приводим данные к “чистому” виду**

``` r
access_points <- access_points %>%
  mutate_at(vars(BSSID, Privacy, Cipher, Authentication, LAN.IP, ESSID),
            trimws) %>%
  mutate_at(vars(BSSID, Privacy, Cipher, Authentication, LAN.IP, ESSID),
            na_if,
            "")

access_points$First.time.seen <-
  as.POSIXct(access_points$First.time.seen, format = "%Y-%m-%d %H:%M:%S")
access_points$Last.time.seen <-
  as.POSIXct(access_points$Last.time.seen, format = "%Y-%m-%d %H:%M:%S")
glimpse(access_points)
```

    Rows: 167
    Columns: 15
    $ BSSID           <chr> "BE:F1:71:D5:17:8B", "6E:C7:EC:16:DA:1A", "9A:75:A8:B9…
    $ First.time.seen <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 …
    $ Last.time.seen  <dttm> 2023-07-28 11:50:50, 2023-07-28 11:55:12, 2023-07-28 …
    $ channel         <int> 1, 1, 1, 7, 6, 6, 11, 11, 11, 1, 6, 14, 11, 11, 6, 6, …
    $ Speed           <int> 195, 130, 360, 360, 130, 130, 195, 130, 130, 195, 180,…
    $ Privacy         <chr> "WPA2", "WPA2", "WPA2", "WPA2", "WPA2", "OPN", "WPA2",…
    $ Cipher          <chr> "CCMP", "CCMP", "CCMP", "CCMP", "CCMP", NA, "CCMP", "C…
    $ Authentication  <chr> "PSK", "PSK", "PSK", "PSK", "PSK", NA, "PSK", "PSK", "…
    $ Power           <int> -30, -30, -68, -37, -57, -63, -27, -38, -38, -66, -42,…
    $ X..beacons      <int> 846, 750, 694, 510, 647, 251, 1647, 1251, 704, 617, 13…
    $ X..IV           <int> 504, 116, 26, 21, 6, 3430, 80, 11, 0, 0, 86, 0, 0, 0, …
    $ LAN.IP          <chr> "0.  0.  0.  0", "0.  0.  0.  0", "0.  0.  0.  0", "0.…
    $ ID.length       <int> 12, 4, 2, 14, 25, 13, 12, 13, 24, 12, 10, 0, 24, 24, 1…
    $ ESSID           <chr> "C322U13 3965", "Cnet", "KC", "POCO X5 Pro 5G", NA, "M…
    $ Key             <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…

``` r
clients <- clients %>%
  mutate_at(vars(Station.MAC, BSSID, Probed.ESSIDs), trimws) %>%
  mutate_at(vars(Station.MAC, BSSID, Probed.ESSIDs), na_if, "")

clients$First.time.seen <-
  as.POSIXct(clients$First.time.seen, format = "%Y-%m-%d %H:%M:%S")
clients$Last.time.seen <-
  as.POSIXct(clients$Last.time.seen, format = "%Y-%m-%d %H:%M:%S")

glimpse(clients)
```

    Rows: 12,269
    Columns: 7
    $ Station.MAC     <chr> "CA:66:3B:8F:56:DD", "96:35:2D:3D:85:E6", "5C:3A:45:9E…
    $ First.time.seen <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 …
    $ Last.time.seen  <dttm> 2023-07-28 10:59:44, 2023-07-28 09:13:03, 2023-07-28 …
    $ Power           <chr> " -33", " -65", " -39", " -61", " -53", " -43", " -31"…
    $ X..packets      <chr> "      858", "        4", "      432", "      958", " …
    $ BSSID           <chr> "BE:F1:71:D5:17:8B", "(not associated)", "BE:F1:71:D6:…
    $ Probed.ESSIDs   <chr> "C322U13 3965", "IT2 Wireless", "C322U21 0566", "C322U…

### Шаг 2: Анализ

#### Точки доступа

1.  **Определить небезопасные точки доступа (без шифрования – OPN)**

``` r
access_points %>%
  filter(Privacy == "OPN") %>%
  select(BSSID) %>%
  distinct(BSSID) %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">BSSID</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:51</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:FF:F2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:74:31</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C8:32</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:50</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:51</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C8:30</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:74:30</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:48:FF:D2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:41</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:40</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E0</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:42</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:40</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:41</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:47:D2</td>
</tr>
<tr class="even">
<td style="text-align: left;">02:BC:15:7E:D5:DC</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B1</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:42</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C8:31</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:47:D1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:AB:0A:00:10:10</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:BD:50</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:0B:B2</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:33:12</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:03:7A:1A:03:56</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:03:7F:12:34:56</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:3E:1A:5D:14:45</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:49:00:B1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:BD:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:EF</td>
</tr>
<tr class="odd">
<td style="text-align: left;">02:67:F1:B0:6C:98</td>
</tr>
<tr class="even">
<td style="text-align: left;">02:CF:8B:87:B4:F9</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:53:7A:99:98:56</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:47:D0</td>
</tr>
</tbody>
</table>

2.  **Определить производителя для каждого обнаруженного устройства**

<!-- -->

    00:03:7A Taiyo Yuden Co., Ltd.
    00:03:7F Atheros Communications, Inc.
    00:25:00 Apple, Inc.
    00:26:99 Cisco Systems, Inc
    E0:D9:E3 Eltex Enterprise Ltd.
    E8:28:C1 Eltex Enterprise Ltd.

``` r
mac_lookup <- data.frame(
  prefix = c(
    "00:03:7A",
    "00:03:7F",
    "00:25:00",
    "00:26:99",
    "E0:D9:E3",
    "E8:28:C1"
  ),
  manufacturer = c(
    "Taiyo Yuden Co., Ltd.",
    "Atheros Communications, Inc.",
    "Apple, Inc.",
    "Cisco Systems, Inc",
    "Eltex Enterprise Ltd.",
    "Eltex Enterprise Ltd."
  )
)

result <- access_points %>%
  mutate(mac_prefix = substr(BSSID, 1, 8)) %>%
  left_join(mac_lookup, by = c("mac_prefix" = "prefix")) %>%
  filter(Privacy == "OPN" & !is.na(manufacturer)) %>%
  select(BSSID, manufacturer)


print(result)
```

                   BSSID                 manufacturer
    1  E8:28:C1:DC:B2:52        Eltex Enterprise Ltd.
    2  E8:28:C1:DC:B2:50        Eltex Enterprise Ltd.
    3  E8:28:C1:DC:B2:51        Eltex Enterprise Ltd.
    4  E8:28:C1:DC:FF:F2        Eltex Enterprise Ltd.
    5  00:25:00:FF:94:73                  Apple, Inc.
    6  E8:28:C1:DD:04:52        Eltex Enterprise Ltd.
    7  E8:28:C1:DE:74:31        Eltex Enterprise Ltd.
    8  E8:28:C1:DE:74:32        Eltex Enterprise Ltd.
    9  E8:28:C1:DC:C8:32        Eltex Enterprise Ltd.
    10 E8:28:C1:DD:04:50        Eltex Enterprise Ltd.
    11 E8:28:C1:DD:04:51        Eltex Enterprise Ltd.
    12 E8:28:C1:DC:C8:30        Eltex Enterprise Ltd.
    13 E8:28:C1:DE:74:30        Eltex Enterprise Ltd.
    14 E0:D9:E3:48:FF:D2        Eltex Enterprise Ltd.
    15 E8:28:C1:DC:B2:41        Eltex Enterprise Ltd.
    16 E8:28:C1:DC:B2:40        Eltex Enterprise Ltd.
    17 00:26:99:F2:7A:E0           Cisco Systems, Inc
    18 E8:28:C1:DC:B2:42        Eltex Enterprise Ltd.
    19 E8:28:C1:DD:04:40        Eltex Enterprise Ltd.
    20 E8:28:C1:DD:04:41        Eltex Enterprise Ltd.
    21 E8:28:C1:DE:47:D2        Eltex Enterprise Ltd.
    22 E8:28:C1:DC:C6:B1        Eltex Enterprise Ltd.
    23 E8:28:C1:DD:04:42        Eltex Enterprise Ltd.
    24 E8:28:C1:DC:C8:31        Eltex Enterprise Ltd.
    25 E8:28:C1:DE:47:D1        Eltex Enterprise Ltd.
    26 E8:28:C1:DC:C6:B0        Eltex Enterprise Ltd.
    27 E8:28:C1:DC:C6:B2        Eltex Enterprise Ltd.
    28 E8:28:C1:DC:BD:50        Eltex Enterprise Ltd.
    29 E8:28:C1:DC:0B:B2        Eltex Enterprise Ltd.
    30 E8:28:C1:DC:33:12        Eltex Enterprise Ltd.
    31 00:03:7A:1A:03:56        Taiyo Yuden Co., Ltd.
    32 00:03:7F:12:34:56 Atheros Communications, Inc.
    33 E0:D9:E3:49:00:B1        Eltex Enterprise Ltd.
    34 E8:28:C1:DC:BD:52        Eltex Enterprise Ltd.
    35 00:26:99:F2:7A:EF           Cisco Systems, Inc
    36 E8:28:C1:DE:47:D0        Eltex Enterprise Ltd.

3.  **Выявить устройства, использующие последнюю версию протокола
    шифрования WPA3, и названия точек доступа, реализованных на этих
    устройствах**

``` r
access_points %>%
  filter(grepl('WPA3', Privacy)) %>%
  select(BSSID, ESSID) %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">BSSID</th>
<th style="text-align: left;">ESSID</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">26:20:53:0C:98:E8</td>
<td style="text-align: left;">NA</td>
</tr>
<tr class="even">
<td style="text-align: left;">A2:FE:FF:B8:9B:C9</td>
<td style="text-align: left;">Christie’s</td>
</tr>
<tr class="odd">
<td style="text-align: left;">96:FF:FC:91:EF:64</td>
<td style="text-align: left;">NA</td>
</tr>
<tr class="even">
<td style="text-align: left;">CE:48:E7:86:4E:33</td>
<td style="text-align: left;">iPhone (Анастасия)</td>
</tr>
<tr class="odd">
<td style="text-align: left;">8E:1F:94:96:DA:FD</td>
<td style="text-align: left;">iPhone (Анастасия)</td>
</tr>
<tr class="even">
<td style="text-align: left;">BE:FD:EF:18:92:44</td>
<td style="text-align: left;">Димасик</td>
</tr>
<tr class="odd">
<td style="text-align: left;">3A:DA:00:F9:0C:02</td>
<td style="text-align: left;">iPhone XS Max 🦊🐱🦊</td>
</tr>
<tr class="even">
<td style="text-align: left;">76:C5:A0:70:08:96</td>
<td style="text-align: left;">NA</td>
</tr>
</tbody>
</table>

4.  **Отсортировать точки доступа по интервалу времени, в течение
    которого они находились на связи, по убыванию.**

``` r
access_points %>%
  mutate(time_online = difftime(Last.time.seen, First.time.seen))%>%
  arrange(desc(time_online)) %>%
  select(BSSID, time_online) %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">BSSID</th>
<th style="text-align: left;">time_online</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">9795 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
<td style="text-align: left;">9776 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
<td style="text-align: left;">9755 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">08:3A:2F:56:35:FE</td>
<td style="text-align: left;">9746 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">6E:C7:EC:16:DA:1A</td>
<td style="text-align: left;">9729 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
<td style="text-align: left;">9726 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:51</td>
<td style="text-align: left;">9725 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">48:5B:39:F9:7A:48</td>
<td style="text-align: left;">9725 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:FF:F2</td>
<td style="text-align: left;">9724 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">8E:55:4A:85:5B:01</td>
<td style="text-align: left;">9723 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
<td style="text-align: left;">9710 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
<td style="text-align: left;">9707 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">1E:93:E3:1B:3C:F4</td>
<td style="text-align: left;">9633 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">9A:75:A8:B9:04:1E</td>
<td style="text-align: left;">9628 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">0C:80:63:A9:6E:EE</td>
<td style="text-align: left;">9628 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:23:EB:E3:81:F2</td>
<td style="text-align: left;">9595 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">9E:A3:A9:DB:7E:01</td>
<td style="text-align: left;">9555 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C8:32</td>
<td style="text-align: left;">9555 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">1C:7E:E5:8E:B7:DE</td>
<td style="text-align: left;">9524 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E1</td>
<td style="text-align: left;">9492 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">BE:F1:71:D5:17:8B</td>
<td style="text-align: left;">9467 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">BE:F1:71:D6:10:D7</td>
<td style="text-align: left;">9461 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">9E:A3:A9:D6:28:3C</td>
<td style="text-align: left;">9451 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:40</td>
<td style="text-align: left;">9400 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:41</td>
<td style="text-align: left;">9400 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:23:EB:E3:81:F1</td>
<td style="text-align: left;">9348 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:23:EB:E3:81:FE</td>
<td style="text-align: left;">9305 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:23:EB:E3:81:FD</td>
<td style="text-align: left;">9305 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">9E:A3:A9:BF:12:C0</td>
<td style="text-align: left;">9270 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:40</td>
<td style="text-align: left;">9212 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">AA:F4:3F:EE:49:0B</td>
<td style="text-align: left;">9045 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:47:D2</td>
<td style="text-align: left;">9041 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:50</td>
<td style="text-align: left;">8989 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">14:EB:B6:6A:76:37</td>
<td style="text-align: left;">8915 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">56:99:98:EE:5A:4E</td>
<td style="text-align: left;">8811 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:42</td>
<td style="text-align: left;">8693 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">38:1A:52:0D:90:A1</td>
<td style="text-align: left;">8661 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">0A:C5:E1:DB:17:7B</td>
<td style="text-align: left;">8608 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C8:30</td>
<td style="text-align: left;">8445 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B1</td>
<td style="text-align: left;">8390 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:42</td>
<td style="text-align: left;">8318 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:41</td>
<td style="text-align: left;">8307 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">12:51:07:FF:29:D6</td>
<td style="text-align: left;">7483 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">CE:B3:FF:84:45:FC</td>
<td style="text-align: left;">7271 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C8:31</td>
<td style="text-align: left;">7199 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
<td style="text-align: left;">6819 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">4A:EC:1E:DB:BF:95</td>
<td style="text-align: left;">6658 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E0</td>
<td style="text-align: left;">6218 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:51</td>
<td style="text-align: left;">5643 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:48:FF:D2</td>
<td style="text-align: left;">5624 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:AB:0A:00:10:10</td>
<td style="text-align: left;">5356 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
<td style="text-align: left;">5190 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">10:50:72:00:11:08</td>
<td style="text-align: left;">4997 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">EA:D8:D1:77:C8:08</td>
<td style="text-align: left;">4995 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">D2:6D:52:61:51:5D</td>
<td style="text-align: left;">4636 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:49:04:52</td>
<td style="text-align: left;">4614 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">7E:3A:10:A7:59:4E</td>
<td style="text-align: left;">4611 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">BE:F1:71:D5:0E:53</td>
<td style="text-align: left;">4578 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">A6:02:B9:73:83:18</td>
<td style="text-align: left;">4577 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">9A:9F:06:44:24:5B</td>
<td style="text-align: left;">4572 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:74:31</td>
<td style="text-align: left;">4433 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">92:F5:7B:43:0B:69</td>
<td style="text-align: left;">4392 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:3C:92</td>
<td style="text-align: left;">4331 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">38:1A:52:0D:84:D7</td>
<td style="text-align: left;">4319 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">38:1A:52:0D:90:5D</td>
<td style="text-align: left;">4255 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">A2:64:E8:97:58:EE</td>
<td style="text-align: left;">4252 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">A6:02:B9:73:81:47</td>
<td style="text-align: left;">4224 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">56:C5:2B:9F:84:90</td>
<td style="text-align: left;">4173 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">A6:02:B9:73:2F:76</td>
<td style="text-align: left;">4144 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">38:1A:52:0D:97:60</td>
<td style="text-align: left;">4086 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">0A:24:D8:D9:24:70</td>
<td style="text-align: left;">4071 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B0</td>
<td style="text-align: left;">3879 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">8A:A3:03:73:52:08</td>
<td style="text-align: left;">3451 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">5E:C7:C0:E4:D7:D4</td>
<td style="text-align: left;">3265 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:54:72</td>
<td style="text-align: left;">3074 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">4A:86:77:04:B7:28</td>
<td style="text-align: left;">3008 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">B6:C4:55:B5:53:24</td>
<td style="text-align: left;">2987 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:BD:50</td>
<td style="text-align: left;">2743 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">76:70:AF:A4:D2:AF</td>
<td style="text-align: left;">2733 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">86:DF:BF:E4:2F:23</td>
<td style="text-align: left;">2688 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">38:1A:52:0D:8F:EC</td>
<td style="text-align: left;">2635 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">EA:7B:9B:D8:56:34</td>
<td style="text-align: left;">2241 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">38:1A:52:0D:85:1D</td>
<td style="text-align: left;">2082 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:CB:AA:62:71</td>
<td style="text-align: left;">1969 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">96:FF:FC:91:EF:64</td>
<td style="text-align: left;">1928 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:33:12</td>
<td style="text-align: left;">1379 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:F0:90</td>
<td style="text-align: left;">1312 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">3A:70:96:C6:30:2C</td>
<td style="text-align: left;">1300 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">36:46:53:81:12:A0</td>
<td style="text-align: left;">1248 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">CE:C3:F7:A4:7E:B3</td>
<td style="text-align: left;">1224 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">26:20:53:0C:98:E8</td>
<td style="text-align: left;">1045 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">92:12:38:E5:7E:1E</td>
<td style="text-align: left;">868 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:33:10</td>
<td style="text-align: left;">846 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DB:F5:F0</td>
<td style="text-align: left;">842 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:0B:B0</td>
<td style="text-align: left;">832 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DB:F5:F2</td>
<td style="text-align: left;">782 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">02:67:F1:B0:6C:98</td>
<td style="text-align: left;">651 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:74:30</td>
<td style="text-align: left;">508 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">1E:C2:8E:D8:30:91</td>
<td style="text-align: left;">498 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">8E:1F:94:96:DA:FD</td>
<td style="text-align: left;">415 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E0:D9:E3:49:04:50</td>
<td style="text-align: left;">401 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">CE:48:E7:86:4E:33</td>
<td style="text-align: left;">295 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:8F</td>
<td style="text-align: left;">288 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">2A:E8:A2:02:01:73</td>
<td style="text-align: left;">220 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2E:FE:13:D0:96:51</td>
<td style="text-align: left;">58 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">9C:A5:13:28:D5:89</td>
<td style="text-align: left;">43 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">22:C9:7F:A9:BA:9C</td>
<td style="text-align: left;">41 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:54:B0</td>
<td style="text-align: left;">36 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">D2:25:91:F6:6C:D8</td>
<td style="text-align: left;">13 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">3A:DA:00:F9:0C:02</td>
<td style="text-align: left;">9 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DB:FC:F2</td>
<td style="text-align: left;">9 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">DC:09:4C:32:34:9B</td>
<td style="text-align: left;">8 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">F2:30:AB:E9:03:ED</td>
<td style="text-align: left;">7 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:49:04:40</td>
<td style="text-align: left;">7 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:03:7A:1A:03:56</td>
<td style="text-align: left;">6 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">B2:CF:C0:00:4A:60</td>
<td style="text-align: left;">5 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">BE:FD:EF:18:92:44</td>
<td style="text-align: left;">4 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">02:BC:15:7E:D5:DC</td>
<td style="text-align: left;">2 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:23:EB:E3:49:31</td>
<td style="text-align: left;">2 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:3E:1A:5D:14:45</td>
<td style="text-align: left;">2 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">76:C5:A0:70:08:96</td>
<td style="text-align: left;">2 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">82:CD:7D:04:17:3B</td>
<td style="text-align: left;">2 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E0:D9:E3:49:00:B0</td>
<td style="text-align: left;">1 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:54:B2</td>
<td style="text-align: left;">1 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">C6:BC:37:7A:67:0D</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">12:48:F9:CF:58:8E</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">76:E4:ED:B0:5C:9A</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:48:FF:D0</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E2:37:BF:8F:6A:7B</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">C2:B5:D7:7F:07:A8</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">8A:4E:75:44:5A:F6</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:03:7A:1A:18:56</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:47:D1</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">A2:FE:FF:B8:9B:C9</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:09:9A:12:55:04</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:3A:B0</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:0B:B2</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:3C:80</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:23:EB:E3:44:31</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">A6:F7:05:31:E8:EE</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">BA:2A:7A:DD:38:3E</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">12:54:1A:C6:FF:71</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">76:5E:F3:F9:A5:1C</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:03:7F:12:34:56</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:03:30</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">B2:1B:0C:67:0A:BD</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E0:D9:E3:49:00:B1</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:BD:52</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:72:D0</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:49:04:41</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F1:1A:E1</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:23:EB:E3:44:32</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:CB:AA:62:72</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:48:B4:D2</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">AE:3E:7F:C8:BC:8E</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">02:B3:45:5A:05:93</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:00:00:00:00:00</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">6A:B0:1A:C2:DF:49</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:3C:90</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">30:B4:B8:11:C0:90</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:EF</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">02:CF:8B:87:B4:F9</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:03:32</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:53:7A:99:98:56</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:03:7F:10:17:56</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:0D:97:6B:93:DF</td>
<td style="text-align: left;">0 secs</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:47:D0</td>
<td style="text-align: left;">0 secs</td>
</tr>
</tbody>
</table>

5.  **Обнаружить топ-10 самых быстрых точек доступа**

``` r
access_points %>% 
  arrange(desc(Speed)) %>%
  select(BSSID, Speed) %>% 
  head(10) %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">BSSID</th>
<th style="text-align: right;">Speed</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">26:20:53:0C:98:E8</td>
<td style="text-align: right;">866</td>
</tr>
<tr class="even">
<td style="text-align: left;">96:FF:FC:91:EF:64</td>
<td style="text-align: right;">866</td>
</tr>
<tr class="odd">
<td style="text-align: left;">CE:48:E7:86:4E:33</td>
<td style="text-align: right;">866</td>
</tr>
<tr class="even">
<td style="text-align: left;">8E:1F:94:96:DA:FD</td>
<td style="text-align: right;">866</td>
</tr>
<tr class="odd">
<td style="text-align: left;">9A:75:A8:B9:04:1E</td>
<td style="text-align: right;">360</td>
</tr>
<tr class="even">
<td style="text-align: left;">4A:EC:1E:DB:BF:95</td>
<td style="text-align: right;">360</td>
</tr>
<tr class="odd">
<td style="text-align: left;">56:C5:2B:9F:84:90</td>
<td style="text-align: right;">360</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:41</td>
<td style="text-align: right;">360</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:40</td>
<td style="text-align: right;">360</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:42</td>
<td style="text-align: right;">360</td>
</tr>
</tbody>
</table>

6.  **Отсортировать точки доступа по частоте отправки запросов (beacons)
    в единицу времени по их убыванию**

``` r
access_points %>%
  mutate(Time = difftime(Last.time.seen, First.time.seen)) %>%
  filter(Time != 0 & X..beacons != 0) %>%
  mutate(BeacPs = X..beacons / as.integer(Time)) %>%
  arrange(desc(BeacPs)) %>%
  select(BSSID, X..beacons, Time, BeacPs) %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">BSSID</th>
<th style="text-align: right;">X..beacons</th>
<th style="text-align: left;">Time</th>
<th style="text-align: right;">BeacPs</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">F2:30:AB:E9:03:ED</td>
<td style="text-align: right;">6</td>
<td style="text-align: left;">7 secs</td>
<td style="text-align: right;">0.8571429</td>
</tr>
<tr class="even">
<td style="text-align: left;">B2:CF:C0:00:4A:60</td>
<td style="text-align: right;">4</td>
<td style="text-align: left;">5 secs</td>
<td style="text-align: right;">0.8000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">3A:DA:00:F9:0C:02</td>
<td style="text-align: right;">5</td>
<td style="text-align: left;">9 secs</td>
<td style="text-align: right;">0.5555556</td>
</tr>
<tr class="even">
<td style="text-align: left;">02:BC:15:7E:D5:DC</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">2 secs</td>
<td style="text-align: right;">0.5000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:3E:1A:5D:14:45</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">2 secs</td>
<td style="text-align: right;">0.5000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">76:C5:A0:70:08:96</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">2 secs</td>
<td style="text-align: right;">0.5000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">D2:25:91:F6:6C:D8</td>
<td style="text-align: right;">5</td>
<td style="text-align: left;">13 secs</td>
<td style="text-align: right;">0.3846154</td>
</tr>
<tr class="even">
<td style="text-align: left;">BE:F1:71:D6:10:D7</td>
<td style="text-align: right;">1647</td>
<td style="text-align: left;">9461 secs</td>
<td style="text-align: right;">0.1740831</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:03:7A:1A:03:56</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">6 secs</td>
<td style="text-align: right;">0.1666667</td>
</tr>
<tr class="even">
<td style="text-align: left;">38:1A:52:0D:84:D7</td>
<td style="text-align: right;">704</td>
<td style="text-align: left;">4319 secs</td>
<td style="text-align: right;">0.1630007</td>
</tr>
<tr class="odd">
<td style="text-align: left;">0A:C5:E1:DB:17:7B</td>
<td style="text-align: right;">1251</td>
<td style="text-align: left;">8608 secs</td>
<td style="text-align: right;">0.1453299</td>
</tr>
<tr class="even">
<td style="text-align: left;">1E:93:E3:1B:3C:F4</td>
<td style="text-align: right;">1390</td>
<td style="text-align: left;">9633 secs</td>
<td style="text-align: right;">0.1442957</td>
</tr>
<tr class="odd">
<td style="text-align: left;">D2:6D:52:61:51:5D</td>
<td style="text-align: right;">647</td>
<td style="text-align: left;">4636 secs</td>
<td style="text-align: right;">0.1395600</td>
</tr>
<tr class="even">
<td style="text-align: left;">BE:F1:71:D5:0E:53</td>
<td style="text-align: right;">617</td>
<td style="text-align: left;">4578 secs</td>
<td style="text-align: right;">0.1347750</td>
</tr>
<tr class="odd">
<td style="text-align: left;">4A:86:77:04:B7:28</td>
<td style="text-align: right;">361</td>
<td style="text-align: left;">3008 secs</td>
<td style="text-align: right;">0.1200133</td>
</tr>
<tr class="even">
<td style="text-align: left;">3A:70:96:C6:30:2C</td>
<td style="text-align: right;">145</td>
<td style="text-align: left;">1300 secs</td>
<td style="text-align: right;">0.1115385</td>
</tr>
<tr class="odd">
<td style="text-align: left;">76:70:AF:A4:D2:AF</td>
<td style="text-align: right;">253</td>
<td style="text-align: left;">2733 secs</td>
<td style="text-align: right;">0.0925723</td>
</tr>
<tr class="even">
<td style="text-align: left;">BE:F1:71:D5:17:8B</td>
<td style="text-align: right;">846</td>
<td style="text-align: left;">9467 secs</td>
<td style="text-align: right;">0.0893631</td>
</tr>
<tr class="odd">
<td style="text-align: left;">AA:F4:3F:EE:49:0B</td>
<td style="text-align: right;">738</td>
<td style="text-align: left;">9045 secs</td>
<td style="text-align: right;">0.0815920</td>
</tr>
<tr class="even">
<td style="text-align: left;">6E:C7:EC:16:DA:1A</td>
<td style="text-align: right;">750</td>
<td style="text-align: left;">9729 secs</td>
<td style="text-align: right;">0.0770891</td>
</tr>
<tr class="odd">
<td style="text-align: left;">4A:EC:1E:DB:BF:95</td>
<td style="text-align: right;">510</td>
<td style="text-align: left;">6658 secs</td>
<td style="text-align: right;">0.0765996</td>
</tr>
<tr class="even">
<td style="text-align: left;">56:C5:2B:9F:84:90</td>
<td style="text-align: right;">317</td>
<td style="text-align: left;">4173 secs</td>
<td style="text-align: right;">0.0759645</td>
</tr>
<tr class="odd">
<td style="text-align: left;">9A:75:A8:B9:04:1E</td>
<td style="text-align: right;">694</td>
<td style="text-align: left;">9628 secs</td>
<td style="text-align: right;">0.0720814</td>
</tr>
<tr class="even">
<td style="text-align: left;">9C:A5:13:28:D5:89</td>
<td style="text-align: right;">3</td>
<td style="text-align: left;">43 secs</td>
<td style="text-align: right;">0.0697674</td>
</tr>
<tr class="odd">
<td style="text-align: left;">36:46:53:81:12:A0</td>
<td style="text-align: right;">82</td>
<td style="text-align: left;">1248 secs</td>
<td style="text-align: right;">0.0657051</td>
</tr>
<tr class="even">
<td style="text-align: left;">38:1A:52:0D:85:1D</td>
<td style="text-align: right;">130</td>
<td style="text-align: left;">2082 secs</td>
<td style="text-align: right;">0.0624400</td>
</tr>
<tr class="odd">
<td style="text-align: left;">38:1A:52:0D:8F:EC</td>
<td style="text-align: right;">107</td>
<td style="text-align: left;">2635 secs</td>
<td style="text-align: right;">0.0406072</td>
</tr>
<tr class="even">
<td style="text-align: left;">2E:FE:13:D0:96:51</td>
<td style="text-align: right;">2</td>
<td style="text-align: left;">58 secs</td>
<td style="text-align: right;">0.0344828</td>
</tr>
<tr class="odd">
<td style="text-align: left;">CE:48:E7:86:4E:33</td>
<td style="text-align: right;">9</td>
<td style="text-align: left;">295 secs</td>
<td style="text-align: right;">0.0305085</td>
</tr>
<tr class="even">
<td style="text-align: left;">8E:1F:94:96:DA:FD</td>
<td style="text-align: right;">12</td>
<td style="text-align: left;">415 secs</td>
<td style="text-align: right;">0.0289157</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:51</td>
<td style="text-align: right;">279</td>
<td style="text-align: left;">9725 secs</td>
<td style="text-align: right;">0.0286889</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
<td style="text-align: right;">260</td>
<td style="text-align: left;">9726 secs</td>
<td style="text-align: right;">0.0267325</td>
</tr>
<tr class="odd">
<td style="text-align: left;">5E:C7:C0:E4:D7:D4</td>
<td style="text-align: right;">85</td>
<td style="text-align: left;">3265 secs</td>
<td style="text-align: right;">0.0260337</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
<td style="text-align: right;">251</td>
<td style="text-align: left;">9755 secs</td>
<td style="text-align: right;">0.0257304</td>
</tr>
<tr class="odd">
<td style="text-align: left;">8E:55:4A:85:5B:01</td>
<td style="text-align: right;">248</td>
<td style="text-align: left;">9723 secs</td>
<td style="text-align: right;">0.0255065</td>
</tr>
<tr class="even">
<td style="text-align: left;">38:1A:52:0D:90:5D</td>
<td style="text-align: right;">90</td>
<td style="text-align: left;">4255 secs</td>
<td style="text-align: right;">0.0211516</td>
</tr>
<tr class="odd">
<td style="text-align: left;">1C:7E:E5:8E:B7:DE</td>
<td style="text-align: right;">142</td>
<td style="text-align: left;">9524 secs</td>
<td style="text-align: right;">0.0149097</td>
</tr>
<tr class="even">
<td style="text-align: left;">38:1A:52:0D:90:A1</td>
<td style="text-align: right;">112</td>
<td style="text-align: left;">8661 secs</td>
<td style="text-align: right;">0.0129315</td>
</tr>
<tr class="odd">
<td style="text-align: left;">A2:64:E8:97:58:EE</td>
<td style="text-align: right;">52</td>
<td style="text-align: left;">4252 secs</td>
<td style="text-align: right;">0.0122295</td>
</tr>
<tr class="even">
<td style="text-align: left;">1E:C2:8E:D8:30:91</td>
<td style="text-align: right;">6</td>
<td style="text-align: left;">498 secs</td>
<td style="text-align: right;">0.0120482</td>
</tr>
<tr class="odd">
<td style="text-align: left;">48:5B:39:F9:7A:48</td>
<td style="text-align: right;">109</td>
<td style="text-align: left;">9725 secs</td>
<td style="text-align: right;">0.0112082</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
<td style="text-align: right;">84</td>
<td style="text-align: left;">9707 secs</td>
<td style="text-align: right;">0.0086535</td>
</tr>
<tr class="odd">
<td style="text-align: left;">38:1A:52:0D:97:60</td>
<td style="text-align: right;">28</td>
<td style="text-align: left;">4086 secs</td>
<td style="text-align: right;">0.0068527</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E1</td>
<td style="text-align: right;">65</td>
<td style="text-align: left;">9492 secs</td>
<td style="text-align: right;">0.0068479</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
<td style="text-align: right;">61</td>
<td style="text-align: left;">9710 secs</td>
<td style="text-align: right;">0.0062822</td>
</tr>
<tr class="even">
<td style="text-align: left;">A6:02:B9:73:2F:76</td>
<td style="text-align: right;">26</td>
<td style="text-align: left;">4144 secs</td>
<td style="text-align: right;">0.0062741</td>
</tr>
<tr class="odd">
<td style="text-align: left;">9E:A3:A9:D6:28:3C</td>
<td style="text-align: right;">51</td>
<td style="text-align: left;">9451 secs</td>
<td style="text-align: right;">0.0053963</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:23:EB:E3:81:FE</td>
<td style="text-align: right;">47</td>
<td style="text-align: left;">9305 secs</td>
<td style="text-align: right;">0.0050510</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:23:EB:E3:81:FD</td>
<td style="text-align: right;">46</td>
<td style="text-align: left;">9305 secs</td>
<td style="text-align: right;">0.0049436</td>
</tr>
<tr class="even">
<td style="text-align: left;">9A:9F:06:44:24:5B</td>
<td style="text-align: right;">22</td>
<td style="text-align: left;">4572 secs</td>
<td style="text-align: right;">0.0048119</td>
</tr>
<tr class="odd">
<td style="text-align: left;">96:FF:FC:91:EF:64</td>
<td style="text-align: right;">9</td>
<td style="text-align: left;">1928 secs</td>
<td style="text-align: right;">0.0046680</td>
</tr>
<tr class="even">
<td style="text-align: left;">A6:02:B9:73:81:47</td>
<td style="text-align: right;">19</td>
<td style="text-align: left;">4224 secs</td>
<td style="text-align: right;">0.0044981</td>
</tr>
<tr class="odd">
<td style="text-align: left;">0C:80:63:A9:6E:EE</td>
<td style="text-align: right;">42</td>
<td style="text-align: left;">9628 secs</td>
<td style="text-align: right;">0.0043623</td>
</tr>
<tr class="even">
<td style="text-align: left;">12:51:07:FF:29:D6</td>
<td style="text-align: right;">32</td>
<td style="text-align: left;">7483 secs</td>
<td style="text-align: right;">0.0042764</td>
</tr>
<tr class="odd">
<td style="text-align: left;">9E:A3:A9:DB:7E:01</td>
<td style="text-align: right;">40</td>
<td style="text-align: left;">9555 secs</td>
<td style="text-align: right;">0.0041863</td>
</tr>
<tr class="even">
<td style="text-align: left;">92:F5:7B:43:0B:69</td>
<td style="text-align: right;">18</td>
<td style="text-align: left;">4392 secs</td>
<td style="text-align: right;">0.0040984</td>
</tr>
<tr class="odd">
<td style="text-align: left;">86:DF:BF:E4:2F:23</td>
<td style="text-align: right;">11</td>
<td style="text-align: left;">2688 secs</td>
<td style="text-align: right;">0.0040923</td>
</tr>
<tr class="even">
<td style="text-align: left;">A6:02:B9:73:83:18</td>
<td style="text-align: right;">17</td>
<td style="text-align: left;">4577 secs</td>
<td style="text-align: right;">0.0037142</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:40</td>
<td style="text-align: right;">30</td>
<td style="text-align: left;">9400 secs</td>
<td style="text-align: right;">0.0031915</td>
</tr>
<tr class="even">
<td style="text-align: left;">26:20:53:0C:98:E8</td>
<td style="text-align: right;">3</td>
<td style="text-align: left;">1045 secs</td>
<td style="text-align: right;">0.0028708</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:42</td>
<td style="text-align: right;">23</td>
<td style="text-align: left;">8318 secs</td>
<td style="text-align: right;">0.0027651</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:41</td>
<td style="text-align: right;">25</td>
<td style="text-align: left;">9400 secs</td>
<td style="text-align: right;">0.0026596</td>
</tr>
<tr class="odd">
<td style="text-align: left;">B6:C4:55:B5:53:24</td>
<td style="text-align: right;">7</td>
<td style="text-align: left;">2987 secs</td>
<td style="text-align: right;">0.0023435</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:50</td>
<td style="text-align: right;">20</td>
<td style="text-align: left;">8989 secs</td>
<td style="text-align: right;">0.0022249</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:23:EB:E3:81:F1</td>
<td style="text-align: right;">19</td>
<td style="text-align: left;">9348 secs</td>
<td style="text-align: right;">0.0020325</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:BD:50</td>
<td style="text-align: right;">5</td>
<td style="text-align: left;">2743 secs</td>
<td style="text-align: right;">0.0018228</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:51</td>
<td style="text-align: right;">9</td>
<td style="text-align: left;">5643 secs</td>
<td style="text-align: right;">0.0015949</td>
</tr>
<tr class="even">
<td style="text-align: left;">02:67:F1:B0:6C:98</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">651 secs</td>
<td style="text-align: right;">0.0015361</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C8:32</td>
<td style="text-align: right;">12</td>
<td style="text-align: left;">9555 secs</td>
<td style="text-align: right;">0.0012559</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C8:31</td>
<td style="text-align: right;">8</td>
<td style="text-align: left;">7199 secs</td>
<td style="text-align: right;">0.0011113</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B0</td>
<td style="text-align: right;">4</td>
<td style="text-align: left;">3879 secs</td>
<td style="text-align: right;">0.0010312</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:CB:AA:62:71</td>
<td style="text-align: right;">2</td>
<td style="text-align: left;">1969 secs</td>
<td style="text-align: right;">0.0010157</td>
</tr>
<tr class="odd">
<td style="text-align: left;">9E:A3:A9:BF:12:C0</td>
<td style="text-align: right;">9</td>
<td style="text-align: left;">9270 secs</td>
<td style="text-align: right;">0.0009709</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C8:30</td>
<td style="text-align: right;">7</td>
<td style="text-align: left;">8445 secs</td>
<td style="text-align: right;">0.0008289</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:23:EB:E3:81:F2</td>
<td style="text-align: right;">7</td>
<td style="text-align: left;">9595 secs</td>
<td style="text-align: right;">0.0007295</td>
</tr>
<tr class="even">
<td style="text-align: left;">7E:3A:10:A7:59:4E</td>
<td style="text-align: right;">3</td>
<td style="text-align: left;">4611 secs</td>
<td style="text-align: right;">0.0006506</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:41</td>
<td style="text-align: right;">5</td>
<td style="text-align: left;">8307 secs</td>
<td style="text-align: right;">0.0006019</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B1</td>
<td style="text-align: right;">5</td>
<td style="text-align: left;">8390 secs</td>
<td style="text-align: right;">0.0005959</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:42</td>
<td style="text-align: right;">5</td>
<td style="text-align: left;">8693 secs</td>
<td style="text-align: right;">0.0005752</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:40</td>
<td style="text-align: right;">5</td>
<td style="text-align: left;">9212 secs</td>
<td style="text-align: right;">0.0005428</td>
</tr>
<tr class="odd">
<td style="text-align: left;">0A:24:D8:D9:24:70</td>
<td style="text-align: right;">2</td>
<td style="text-align: left;">4071 secs</td>
<td style="text-align: right;">0.0004913</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:74:31</td>
<td style="text-align: right;">2</td>
<td style="text-align: left;">4433 secs</td>
<td style="text-align: right;">0.0004512</td>
</tr>
<tr class="odd">
<td style="text-align: left;">EA:7B:9B:D8:56:34</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">2241 secs</td>
<td style="text-align: right;">0.0004462</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
<td style="text-align: right;">4</td>
<td style="text-align: left;">9776 secs</td>
<td style="text-align: right;">0.0004092</td>
</tr>
<tr class="odd">
<td style="text-align: left;">10:50:72:00:11:08</td>
<td style="text-align: right;">2</td>
<td style="text-align: left;">4997 secs</td>
<td style="text-align: right;">0.0004002</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:47:D2</td>
<td style="text-align: right;">3</td>
<td style="text-align: left;">9041 secs</td>
<td style="text-align: right;">0.0003318</td>
</tr>
<tr class="odd">
<td style="text-align: left;">EA:D8:D1:77:C8:08</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">4995 secs</td>
<td style="text-align: right;">0.0002002</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">5190 secs</td>
<td style="text-align: right;">0.0001927</td>
</tr>
<tr class="odd">
<td style="text-align: left;">56:99:98:EE:5A:4E</td>
<td style="text-align: right;">1</td>
<td style="text-align: left;">8811 secs</td>
<td style="text-align: right;">0.0001135</td>
</tr>
</tbody>
</table>

#### Данные клиентов

1.  **Определить производителя для каждого обнаруженного устройства**

``` r
mac_lookup <- data.frame(
  prefix = c(
    "00:03:7F",
    "00:0D:97",
    "00:23:EB",
    "00:25:00",
    "00:26:99",
    "08:3A:2F",
    "0C:80:63",
    "DC:09:4C",
    "E0:D9:E3",
    "E8:28:C1"
  ),
  manufacturer = c(
    "Atheros Communications, Inc.",
    "Hitachi Energy USA Inc.",
    "Cisco Systems, Inc",
    "Apple, Inc.",
    "Cisco Systems, Inc",
    "Guangzhou Juan Intelligent Tech Joint Stock Co.,Ltd",
    "Tp-Link Technologies Co.,Ltd.",
    "Huawei Technologies Co.,Ltd",
    "Eltex Enterprise Ltd.",
    "Eltex Enterprise Ltd."
  )
)

result <- clients %>%
  filter(BSSID != "(not associated)")%>%
  filter(!is.na(BSSID)) %>%
  arrange(BSSID) %>%
  unique() %>%
  mutate(mac_prefix = substr(BSSID, 1, 8)) %>%
  left_join(mac_lookup, by = c("mac_prefix" = "prefix")) %>%
  filter(!is.na(manufacturer)) %>%
  select(BSSID, manufacturer) %>%
  knitr::kable()

result
```

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 74%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">BSSID</th>
<th style="text-align: left;">manufacturer</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">00:03:7F:10:17:56</td>
<td style="text-align: left;">Atheros Communications, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:0D:97:6B:93:DF</td>
<td style="text-align: left;">Hitachi Energy USA Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:23:EB:E3:49:31</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">Apple, Inc.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:BA:75:80</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:BA:75:80</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:BA:75:80</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:BA:75:80</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E1</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
<td style="text-align: left;">Cisco Systems, Inc</td>
</tr>
<tr class="odd">
<td style="text-align: left;">08:3A:2F:56:35:FE</td>
<td style="text-align: left;">Guangzhou Juan Intelligent Tech Joint
Stock Co.,Ltd</td>
</tr>
<tr class="even">
<td style="text-align: left;">08:3A:2F:56:35:FE</td>
<td style="text-align: left;">Guangzhou Juan Intelligent Tech Joint
Stock Co.,Ltd</td>
</tr>
<tr class="odd">
<td style="text-align: left;">08:3A:2F:56:35:FE</td>
<td style="text-align: left;">Guangzhou Juan Intelligent Tech Joint
Stock Co.,Ltd</td>
</tr>
<tr class="even">
<td style="text-align: left;">08:3A:2F:56:35:FE</td>
<td style="text-align: left;">Guangzhou Juan Intelligent Tech Joint
Stock Co.,Ltd</td>
</tr>
<tr class="odd">
<td style="text-align: left;">0C:80:63:A9:6E:EE</td>
<td style="text-align: left;">Tp-Link Technologies Co.,Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">0C:80:63:A9:6E:EE</td>
<td style="text-align: left;">Tp-Link Technologies Co.,Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">DC:09:4C:32:34:9B</td>
<td style="text-align: left;">Huawei Technologies Co.,Ltd</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:49:04:40</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E0:D9:E3:49:04:41</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:49:04:52</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DB:F5:F2</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:0B:B0</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:33:10</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:33:12</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:3C:92</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:54:B2</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:40</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:41</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C8:30</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C8:32</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C8:32</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:F0:90</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:F0:90</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:F0:90</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:FF:F2</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:FF:F2</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:FF:F2</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:40</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:40</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:41</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:50</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:50</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
<td style="text-align: left;">Eltex Enterprise Ltd.</td>
</tr>
</tbody>
</table>

2.  **Обнаружить устройства, которые НЕ рандомизируют свой MAC адрес**

``` r
clients  %>%
  select(BSSID) %>%
  filter(BSSID != "(not associated)" &
           !grepl("^.[2|6|A|E].+", BSSID)) %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">BSSID</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:FF:F2</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
</tr>
<tr class="even">
<td style="text-align: left;">0C:80:63:A9:6E:EE</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">08:3A:2F:56:35:FE</td>
</tr>
<tr class="odd">
<td style="text-align: left;">08:3A:2F:56:35:FE</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
</tr>
<tr class="even">
<td style="text-align: left;">08:3A:2F:56:35:FE</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C8:32</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:50</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:FF:F2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:3C:92</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">0C:80:63:A9:6E:EE</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="even">
<td style="text-align: left;">08:3A:2F:56:35:FE</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:AB:0A:00:10:10</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C8:30</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">MIREA_HOTSPOT</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DB:F5:F2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:FF:F2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:40</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:23:EB:E3:49:31</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:40</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E0:D9:E3:49:04:40</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:41</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:40</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:33:12</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="even">
<td style="text-align: left;">AndroidShare_6329</td>
</tr>
<tr class="odd">
<td style="text-align: left;">DC:09:4C:32:34:9B</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:F0:90</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:49:04:52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:F0:90</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:41</td>
</tr>
<tr class="even">
<td style="text-align: left;">TP-Link_9872_5G</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:33:10</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:49:04:41</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:50</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:F0:90</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C8:32</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:0B:B0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:54:B2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:03:7F:10:17:56</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:0D:97:6B:93:DF</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
</tbody>
</table>

3.  **Кластеризовать запросы от устройств к точкам доступа по их именам.
    Определить время появления устройства в зоне радиовидимости и время
    выхода его из нее**

``` r
clients %>%
  filter(!is.na(Probed.ESSIDs)) %>%
  group_by(Probed.ESSIDs) %>%
  summarise("min time" = min(First.time.seen),
            "max time" = max(Last.time.seen),)
```

    # A tibble: 107 × 3
       Probed.ESSIDs                `min time`          `max time`         
       <chr>                        <dttm>              <dttm>             
     1 -D-13-                       2023-07-28 09:14:42 2023-07-28 10:26:42
     2 1                            2023-07-28 10:36:12 2023-07-28 11:56:13
     3 107                          2023-07-28 10:29:43 2023-07-28 10:29:43
     4 531                          2023-07-28 10:57:04 2023-07-28 10:57:04
     5 AAAAAOB/CC0ADwGkRedmi 3S     2023-07-28 09:34:20 2023-07-28 11:44:40
     6 AKADO-D967                   2023-07-28 10:31:55 2023-07-28 10:31:55
     7 AQAAAB6zaIoATwEURedmi Note 5 2023-07-28 10:25:19 2023-07-28 11:51:48
     8 ASUS                         2023-07-28 10:31:13 2023-07-28 10:31:13
     9 Alex-net2                    2023-07-28 10:01:06 2023-07-28 10:01:06
    10 AndroidAP177B                2023-07-28 09:13:09 2023-07-28 11:34:42
    # ℹ 97 more rows

4.  **Оценить стабильность уровня сигнала внури кластера во времени.
    Выявить наиболее стабильный кластер.**

``` r
clients %>%
  mutate(t = as.integer(difftime(Last.time.seen, First.time.seen))) %>%
  filter(t != 0 &
           !is.na(Probed.ESSIDs)) %>%
  group_by(Probed.ESSIDs) %>%
  summarise(Mean = mean(t), Sd = sd(t)) %>%
  filter(!is.na(Sd) & 
           Sd != 0) %>%
  arrange(Sd, Mean) %>%
  select(Probed.ESSIDs, Mean, Sd) %>%
  head(1)
```

    # A tibble: 1 × 3
      Probed.ESSIDs  Mean    Sd
      <chr>         <dbl> <dbl>
    1 nvripcsuite    9780  3.46

## ️Оценка результата

Развиты практические навыки использования функций обработки данных
пакета dplyr – функции select(), filter(), mutate(), arrange(),
group_by()

## ️Вывод

1.  Загружен датасет
2.  Выполнены задания
3.  Составлен отчет
