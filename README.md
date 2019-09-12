
  - [rOstluft.plot](#rostluft.plot)
  - [Installation](#installation)
  - [Beispiele](#beispiele)
      - [Windrose auf Karte](#windrose-auf-karte)
      - [Radar-chart Windstatistik](#radar-chart-windstatistik)
      - [polarplot openair-style](#polarplot-openair-style)
      - [Tagesgang-Jahresgang heatmap](#tagesgang-jahresgang-heatmap)
      - [Kalender + stat\_filter](#kalender-stat_filter)
      - [Hysplit Trajektorien (openair data
        format)](#hysplit-trajektorien-openair-data-format)
      - [Squishing data](#squishing-data)
      - [padding data](#padding-data)

<!-- README.md is generated from README.Rmd. Please edit that file -->

<img src="man/figures/logo.png" align="right" />

# rOstluft.plot

Erstellen von Diagrammen für Ostluft Auswertungen und Berichte mit Bezug
zu Luftschadstoffen und Meteorologie. Einige Funktionen sind aus dem
package [openair](http://www.openair-project.org) abgeleitet. Alle
plot-Funktionen sind grundsätzlich auf das
[ggplot2](https://ggplot2.tidyverse.org) package bezogen.

# Installation

Der Quellcode von
[rOstluft.plot](https://github.com/Ostluft/rOstluft.plot) ist auf github
gehosted. Die einfachste Variante ist die Installation mit Hilfe des
Packages devtools:

``` r
#install.packages("devtools")
devtools::install_github("Ostluft/rOstluft.plot")
```

# Beispiele

``` r
library(ggplot2)
library(rOstluft.plot)
library(dplyr)

data <-
  rOstluft::read_airmo_csv(system.file("extdata", "Zch_Stampfenbachstrasse_2010-2014.csv", package = "rOstluft.data", mustWork = TRUE)) %>%
  rOstluft::rolf_to_openair() %>%
  openair::cutData(date, type = "daylight")
```

## Windrose auf Karte

``` r
bb <- bbox_lv95(2683141, 1249040, 500)
bg <- get_stamen_map(bb)

ggwindrose(data, ws, wd, ws_max = 4, bg = bg) +
  theme(
    panel.grid.major = element_line(linetype = 2, color = "black", size = 0.5)
   )
```

<img src="man/figures/README-unnamed-chunk-3-1.png" width="100%" />

``` r
groupings = rOstluft.plot::groups(daylight)

# the facetting variable has to be in the groupings argument
ggwindrose(data, ws, wd, ws_max = 4, groupings = groupings) +
   facet_wrap(vars(daylight))
```

<img src="man/figures/README-unnamed-chunk-3-2.png" width="100%" />

## Radar-chart Windstatistik

``` r
# df <-
#   rOstluft::read_airmo_csv(system.file("extdata", "Zch_Stampfenbachstrasse_2010-2014.csv",package = "rOstluft.data", mustWork = TRUE)) %>%
#   rOstluft::rolf_to_openair() %>%
#   dplyr::mutate(wday = lubridate::wday(date, label = TRUE, week_start = 1))
# 
# ggradar(df, aes(wd = wd, ws = ws, z = NOx), 
#         param_args = list(fill = "blue", color = "blue", alpha = 0.5)) + ylab("NOx")
# 
# df <- openair::cutData(df, date, type = "daylight") %>% 
#   dplyr::select(wd, ws, NO, NOx, daylight) %>% 
#   tidyr::gather(par, val, -wd, -ws, -daylight)
#  
# ggradar(df, aes(wd = wd, ws = ws, z = val, group = par, fill = par, color = par)) + ylab("mean") +
#   facet_wrap(daylight~.)
```

## polarplot openair-style

``` r
fill_scale = scale_fill_gradientn(colours = alpha(matlab::jet.colors(100), 0.75), na.value = NA)
ggpolarplot(data, wd = wd, ws = ws, z = NOx, bg = bg, 
            fill_scale = fill_scale, breaks = seq(0,8,2))
```

<img src="man/figures/README-unnamed-chunk-5-1.png" width="100%" />

## Tagesgang-Jahresgang heatmap

``` r
ggyearday(data, time = "date", z = "O3")
```

<img src="man/figures/README-unnamed-chunk-6-1.png" width="100%" />

## Kalender + stat\_filter

Kalender der max Stundenwerte des Tages von Ozon

``` r
statstable <- tibble::tribble(
  ~parameter, ~statistic, ~from, ~to,
  "O3", "mean", "input", "h1",
  "O3", "max", "h1", "d1"
)

data_d1 <- 
  rOstluft.data::f("Zch_Stampfenbachstrasse_2010-2014.csv") %>% 
  rOstluft::read_airmo_csv() %>%
  dplyr::filter(starttime < lubridate::ymd(20130101)) %>% 
  rOstluft::calculate_statstable(statstable) %>%
  purrr::pluck("d1") %>% 
  rOstluft::rolf_to_openair()

  
ggcalendar(data_d1, z = "O3_max_h1") +
  scale_fill_viridis_c(direction = -1, option = "magma", na.value = NA) +
  cal_month_border(color = "black") +
  stat_filter(
    aes(filter = O3_max_h1 > 120), size = 1, 
    color = "white", fill = "white", shape = 21,
    position = position_nudge(y = 0.25)
  ) +
  cal_label(aes(label = round(O3_max_h1,0)), fontface = "bold")
```

<img src="man/figures/README-unnamed-chunk-7-1.png" width="100%" />

## Hysplit Trajektorien (openair data format)

``` r
fn <- system.file("extdata", "2017_ZH-Kaserne-hysplit.rds", package = "rOstluft.data")
traj <- readRDS(fn)
traj <- dplyr::filter(traj, date < lubridate::ymd("2017-01-08"))

ggtraj(traj, color_scale = ggplot2::scale_color_viridis_c(name = "m agl."))
```

<img src="man/figures/README-unnamed-chunk-8-1.png" width="100%" />

## Squishing data

Messdaten enthalten oft Extremwerte von ausserordentlichen Episoden oder
Ereignissen. Als Beispiel Feuwerwerke oder Inversionen in den PM10
Daten:

``` r
ggyearday(data, time = date, z = PM10)
```

<img src="man/figures/README-unnamed-chunk-9-1.png" width="100%" />

In einem ggplot2 Diagramm kann bei continuous scales mit Hilfe dem
Argument `oob` eine Funktion übergeben werden, was mit Werten ausserhalb
des Limits geschieht. Mit Hilfe der Funktion
[`scales::squish()`](https://scales.r-lib.org/reference/squish.html)
werden diese Werte auf das Minima, bzw. Maxima der Limits gesetzt. In
rOstluft.plot sind die Hilfsfunktionen `scale_fill_viridis_squished()`,
`scale_color_viridis_squished()`, `scale_fill_gradientn_squished()` und
`scale_color_gradientn_squished()` enthalten:

``` r
fill_scale <- scale_fill_viridis_squished(
  breaks=c(0, 20, 40, 60, 80), 
  limits = c(0, 80),
  direction = -1, 
  na.value = NA, 
  option = "A"
)

ggyearday(data, time = date, z = PM10, fill_scale = fill_scale)
```

<img src="man/figures/README-unnamed-chunk-10-1.png" width="100%" />

Teilweise ist es für Klassierungen praktisch alle Werte über einem
Maximum in einer zusätzlichen Klasse zusammen zu fassen. Die Funktion
`cut_ws()` beinhaltet diese Funktionalität, hat jedoch gewisse
Einschränkungen (Negative Werte werden zu NA, Breite der Klasse fix):

``` r
pm10_right <- cut_ws(data$PM10, binwidth = 20, ws_max = 80)
table(pm10_right)
```

    #> pm10_right
    #>  [0,20] (20,40] (40,60] (60,80]     >80 
    #>   48146   27874    6162    1191     476

``` r
pm10_left <- cut_ws(data$PM10, 20, 80, right = FALSE)

# bei der Umwandlung der Ausgabe nach HTML wird "≥80" in "=80" 
# umgewandelt. In Diagrammen und der R Konsole wird das Zeichen
# jedoch korrekt dargestellt. See https://github.com/r-lib/evaluate/issues/59
table(pm10_left)
```

    #> pm10_left
    #>  [0,20) [20,40) [40,60) [60,80)     =80 
    #>   48146   27874    6162    1191     476

Für mehr Flexibilät kann direkt `base::cut()` verwendet werden und
breaks mit `-Inf` und `Inf` definiert werden.

``` r
breaks <- c(-Inf, 0, 19, 41, 66, 80, Inf)
pm10_cut <- cut(data$PM10, breaks = breaks, right = TRUE, include.lowest = TRUE)
table(pm10_cut)
```

    #> pm10_cut
    #>  [-Inf,0]    (0,19]   (19,41]   (41,66]   (66,80] (80, Inf] 
    #>      1141     45516     31079      6164       614       476

## padding data

Messdaten liegen nicht immer in vollständigen Zeitreihen vor. Für einige
Diagramme ist es jedoch erforderlich, dass für alle Zeitpunkte ein Wert
oder ein NA vorhanden ist. Für Daten im rolf Format können die rOstluft
Funktionen `rOstluft::pad()` und `rOstluft::pad_year()` verwenden
werden. rOstluft.plot enthält 2 generische padding Funktionen:

``` r
fn <- rOstluft.data::f("Zch_Stampfenbachstrasse_min30_2013_Jan.csv")
january <- rOstluft::read_airmo_csv(fn)
january_oa <- rOstluft::rolf_to_openair(january
                                        )
tail(january_oa)
```

    #> # A tibble: 6 x 16
    #>   date                site     CO    Hr    NO   NO2   NOx    O3     p  PM10
    #>   <dttm>              <fct> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
    #> 1 2013-01-31 21:00:00 Zch_~ 0.191  67.3 0.675  7.70  4.57  71.0  970.  7.21
    #> 2 2013-01-31 21:30:00 Zch_~ 0.195  64.9 0.359  7.72  4.33  69.7  970.  4.89
    #> 3 2013-01-31 22:00:00 Zch_~ 0.191  65.1 0.424  6.84  3.92  69.0  970.  6.71
    #> 4 2013-01-31 22:30:00 Zch_~ 0.184  67.3 0.353  5.38  3.09  70.5  970.  5.19
    #> 5 2013-01-31 23:00:00 Zch_~ 0.186  67.3 0.634  5.87  3.58  70.2  969.  5.79
    #> 6 2013-01-31 23:30:00 Zch_~ 0.189  68.7 0.435  6.76  3.88  67.6  969.  7.92
    #> # ... with 6 more variables: RainDur <dbl>, SO2 <dbl>, StrGlo <dbl>,
    #> #   T <dbl>, wd <dbl>, ws <dbl>

``` r
# site mit "Zch_Stampfenbachstrasse" füllen
pad_to_year(january_oa, date, "30 min", fill = list(site = "Zch_Stampfenbachstrasse")) %>% 
  tail()
```

    #> # A tibble: 6 x 16
    #>   date                site     CO    Hr    NO   NO2   NOx    O3     p  PM10
    #>   <dttm>              <fct> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
    #> 1 2013-12-31 21:00:00 Zch_~    NA    NA    NA    NA    NA    NA    NA    NA
    #> 2 2013-12-31 21:30:00 Zch_~    NA    NA    NA    NA    NA    NA    NA    NA
    #> 3 2013-12-31 22:00:00 Zch_~    NA    NA    NA    NA    NA    NA    NA    NA
    #> 4 2013-12-31 22:30:00 Zch_~    NA    NA    NA    NA    NA    NA    NA    NA
    #> 5 2013-12-31 23:00:00 Zch_~    NA    NA    NA    NA    NA    NA    NA    NA
    #> 6 2013-12-31 23:30:00 Zch_~    NA    NA    NA    NA    NA    NA    NA    NA
    #> # ... with 6 more variables: RainDur <dbl>, SO2 <dbl>, StrGlo <dbl>,
    #> #   T <dbl>, wd <dbl>, ws <dbl>

``` r
# automatisch alle factor/character columns füllen
pad_to_year_fill(january_oa, date, "30 min") %>% 
  tail()
```

    #> # A tibble: 6 x 16
    #>   date                site     CO    Hr    NO   NO2   NOx    O3     p  PM10
    #>   <dttm>              <fct> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
    #> 1 2013-12-31 21:00:00 Zch_~    NA    NA    NA    NA    NA    NA    NA    NA
    #> 2 2013-12-31 21:30:00 Zch_~    NA    NA    NA    NA    NA    NA    NA    NA
    #> 3 2013-12-31 22:00:00 Zch_~    NA    NA    NA    NA    NA    NA    NA    NA
    #> 4 2013-12-31 22:30:00 Zch_~    NA    NA    NA    NA    NA    NA    NA    NA
    #> 5 2013-12-31 23:00:00 Zch_~    NA    NA    NA    NA    NA    NA    NA    NA
    #> 6 2013-12-31 23:30:00 Zch_~    NA    NA    NA    NA    NA    NA    NA    NA
    #> # ... with 6 more variables: RainDur <dbl>, SO2 <dbl>, StrGlo <dbl>,
    #> #   T <dbl>, wd <dbl>, ws <dbl>

``` r
pad_to_year_fill(january, starttime, "30 min") %>% 
  tail()
```

    #> # A tibble: 6 x 6
    #>   starttime           site                   parameter interval unit  value
    #>   <dttm>              <fct>                  <fct>     <fct>    <fct> <dbl>
    #> 1 2013-12-31 21:00:00 Zch_Stampfenbachstras~ WVv       min30    m/s      NA
    #> 2 2013-12-31 21:30:00 Zch_Stampfenbachstras~ WVv       min30    m/s      NA
    #> 3 2013-12-31 22:00:00 Zch_Stampfenbachstras~ WVv       min30    m/s      NA
    #> 4 2013-12-31 22:30:00 Zch_Stampfenbachstras~ WVv       min30    m/s      NA
    #> 5 2013-12-31 23:00:00 Zch_Stampfenbachstras~ WVv       min30    m/s      NA
    #> 6 2013-12-31 23:30:00 Zch_Stampfenbachstras~ WVv       min30    m/s      NA

``` r
# enthalten die Daten jedoch eine Klassifizierungs Spalte
# muss man die zu füllenden Spalten explixit angeben

january_oa <- openair::cutData(january_oa, "month") %>% 
  dplyr::select(date, month, dplyr::everything())

# Monats Spalte wird falscherweise mit Januar gefüllt
# Ausserdem würden für jeden Monat die Daten multipliziert
pad_to_year_fill(january_oa, date, "30 min") %>% 
  tail()
```

    #> # A tibble: 6 x 17
    #>   date                month site     CO    Hr    NO   NO2   NOx    O3     p
    #>   <dttm>              <ord> <fct> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
    #> 1 2013-12-31 21:00:00 Janu~ Zch_~    NA    NA    NA    NA    NA    NA    NA
    #> 2 2013-12-31 21:30:00 Janu~ Zch_~    NA    NA    NA    NA    NA    NA    NA
    #> 3 2013-12-31 22:00:00 Janu~ Zch_~    NA    NA    NA    NA    NA    NA    NA
    #> 4 2013-12-31 22:30:00 Janu~ Zch_~    NA    NA    NA    NA    NA    NA    NA
    #> 5 2013-12-31 23:00:00 Janu~ Zch_~    NA    NA    NA    NA    NA    NA    NA
    #> 6 2013-12-31 23:30:00 Janu~ Zch_~    NA    NA    NA    NA    NA    NA    NA
    #> # ... with 7 more variables: PM10 <dbl>, RainDur <dbl>, SO2 <dbl>,
    #> #   StrGlo <dbl>, T <dbl>, wd <dbl>, ws <dbl>

``` r
# mit explixiter Defintion der zu füllenden Spalten klappt es
pad_to_year_fill(january_oa, date, "30 min", site) %>% 
  tail()
```

    #> # A tibble: 6 x 17
    #>   date                month site     CO    Hr    NO   NO2   NOx    O3     p
    #>   <dttm>              <ord> <fct> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
    #> 1 2013-12-31 21:00:00 <NA>  Zch_~    NA    NA    NA    NA    NA    NA    NA
    #> 2 2013-12-31 21:30:00 <NA>  Zch_~    NA    NA    NA    NA    NA    NA    NA
    #> 3 2013-12-31 22:00:00 <NA>  Zch_~    NA    NA    NA    NA    NA    NA    NA
    #> 4 2013-12-31 22:30:00 <NA>  Zch_~    NA    NA    NA    NA    NA    NA    NA
    #> 5 2013-12-31 23:00:00 <NA>  Zch_~    NA    NA    NA    NA    NA    NA    NA
    #> 6 2013-12-31 23:30:00 <NA>  Zch_~    NA    NA    NA    NA    NA    NA    NA
    #> # ... with 7 more variables: PM10 <dbl>, RainDur <dbl>, SO2 <dbl>,
    #> #   StrGlo <dbl>, T <dbl>, wd <dbl>, ws <dbl>
