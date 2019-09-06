
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

ggwindrose(data, ws, wd, ws_max = 4, bg = bg)
```

<img src="man/figures/README-unnamed-chunk-4-1.png" width="100%" />

``` r
groupings = rOstluft.plot::groups(daylight)

# the facetting variable has to be in the groupings argument
ggwindrose(data, ws, wd, ws_max = 4, groupings = groupings) +
   facet_wrap(vars(daylight))
```

<img src="man/figures/README-unnamed-chunk-4-2.png" width="100%" />

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

<img src="man/figures/README-unnamed-chunk-6-1.png" width="100%" />

## Tagesgang-Jahresgang heatmap

``` r
ggyearday(data, time = "date", z = "O3")
```

<img src="man/figures/README-unnamed-chunk-7-1.png" width="100%" />

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

<img src="man/figures/README-unnamed-chunk-8-1.png" width="100%" />

## Hysplit Trajektorien (openair data format)

``` r
fn <- system.file("extdata", "2017_ZH-Kaserne-hysplit.rds", package = "rOstluft.data")
traj <- readRDS(fn)
traj <- dplyr::filter(traj, date < lubridate::ymd("2017-01-08"))

ggtraj(traj, color_scale = ggplot2::scale_color_viridis_c(name = "m agl."))
```

<img src="man/figures/README-unnamed-chunk-9-1.png" width="100%" />
