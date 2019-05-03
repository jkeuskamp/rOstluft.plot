
<!-- README.md is generated from README.Rmd. Please edit that file -->
<img src="man/figures/logo.png" align="right" />

rOstluft.plot
=============

Erstellen von Diagrammen für Ostluft Auswertungen und Berichte

Installation
============

Der Quellcode von [rOstluft.plot](https://github.com/Ostluft/rOstluft.plot) ist auf github gehosted. Die einfachste Variante ist die Installation mit Hilfe des Packages devtools:

``` r
#install.packages("devtools")
devtools::install_github("Ostluft/rOstluft.plot")
```

Beispiele
=========

Hysplit Trajektorien:
---------------------

``` r
library(ggplot2)
library(rOstluft)
library(rOstluft.plot)

fn <- system.file("extdata", "2017_ZH-Kaserne-hysplit.rds", package = "rOstluft.data")
traj <- readRDS(fn)
traj <- dplyr::filter(traj, date < lubridate::ymd("2017-01-08"))
hysplit_traj(traj, color_scale = ggplot2::scale_color_viridis_c(name = "m agl."))
```

<img src="man/figures/README-unnamed-chunk-2-1.png" width="100%" />

Kalender + stat\_filter
-----------------------

Kalender der max Stundenwerte des Tages von Ozon

``` r
store <- storage_s3_rds("aqmet", format = format_rolf(), bucket = "rostluft", prefix = "aqmet")
data <- store$get(site = "Zch_Stampfenbachstrasse", year = 2016:2017, interval = "d1")
data <- rolf_to_openair(data)

plt_cal(data, date, O3_max_h1) + 
  scale_fill_viridis_c(name = "max. O3 Stundenmittel", end = 0.9, option = "magma") +
  cal_month_border(size = 2) +
  cal_label(aes(label = round(O3_max_h1)), position = position_nudge(y = -0.1), fontface = "bold", size = 3) +
  stat_filter(aes(filter = O3_max_h1 > 120), position = position_nudge(y = 0.2), size = 2) +
  theme(legend.position = "top")
```

<img src="man/figures/README-unnamed-chunk-3-1.png" width="100%" />
