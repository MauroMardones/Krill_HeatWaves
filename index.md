---
title: "Correlation analysis Krill recriutment and environmental covariables"
subtitle: "Stock assessment krill populations supplementary methods"
author: "Mardones, M; Rebolledo, L. ; Krugger, L."
date:  "06 July, 2023"
bibliography: heatwave.bib
csl: apa.csl
link-citations: yes
linkcolor: blue
output:
  html_document:
    keep_md: true
    toc: true
    toc_deep: 3
    toc_float:
      collapsed: false
      smooth_scroll: false
    theme: cosmo
    fontsize: 0.9em
    linestretch: 1.7
    html-math-method: katex
    self-contained: true
    code-tools: true
editor_options: 
  markdown: 
    wrap: 72
---


```r
rm(list = ls())
knitr::opts_chunk$set(echo = TRUE,
                      message = FALSE,
                      warning = FALSE,
                      fig.align = 'center',
                      dev = 'jpeg',
                      dpi = 300)
#XQuartz is a mess, put this in your onload to default to cairo instead
options(bitmapType = "cairo") 
# (https://github.com/tidyverse/ggplot2/issues/2655)
# Lo mapas se hacen mas rapido
```


```r
library(reshape2)
library(tidyverse)
library(plyr)
library(lubridate)
library(raster)
library(sf)
library(CCAMLRGIS)
library(here)
library(easystats)
library(see) # for plotting
library(ggraph) # needs to be loaded# analisis estadisticos post
library(ggridges)
library(ggpubr)
library(knitr)
library(kableExtra)
# stat test
library(corrplot)
library(outliers)
library(visreg)
# correlation test 
library(PerformanceAnalytics)
library(psych)
library(lme4)
library(sjPlot)
```

# Background

The following document intends to carry out a complementary
methodological analysis to correlate environmental variables with the
population dynamics of krill (*Euphausia superba*), in this case, with a biological component like lengths from fishery monitoring.

The main idea is join length and covariabes into strata polygons.

# Packages used 

@heatwavesR, @reraddp2023

# Secuencia analysis

- Cargar shapefiles de los stratas
- Cargar datos Ambientales `dataenvf`
- Cargar datos de de longitud krill `sf4`


```r
load("datgeo5.RData")
load("datchl.RData")
load("dattsm.RData")
load("~/DOCAS/Data/KrillLows/DataEnvKrill.RData")
```

change lat-long format to `sf`.


```r
#Transformo
#Hielo
gsic <- st_as_sf(datgeo5, coords = c("longitud", "latitud"),  
                  crs = "+proj=latlong +ellps=WGS84") 
# TSM
gtsm <- st_as_sf(dattsm , coords = c("lon", "lat"),  
                  crs = "+proj=latlong +ellps=WGS84")

gchl <- st_as_sf(datchl, coords = c("lon", "lat"),  
                  crs = "+proj=latlong +ellps=WGS84")
```



Join


```r
joinenv <- list(as.data.frame(gsic),
           as.data.frame(gtsm),
           as.data.frame(gchl)) %>% 
        reduce(full_join, by='geometry', 'ANO')

joinenv2 <- list(as.data.frame(gsic),
           as.data.frame(gtsm),
           as.data.frame(gchl)) %>% 
        reduce(full_join, by='geometry')
```




```r
strata <- st_read("Strata.shp", quiet=T)
strata=st_transform(strata, "+proj=latlong +ellps=WGS84")
```



```r
stratasub <- st_read("Clipped_Strata.shp", quiet=T)
stratasub <- st_transform(stratasub, "+proj=latlong +ellps=WGS84")
```



plot simple 


```r
ssmap <- ggplot()+
  geom_sf(data = stratasub, aes(fill=stratasub$ID, 
                           alpha=0.3))+
  # geom_sf(data = ssmu481aa, aes(fill=ssmu481aa$GAR_Short_Label, 
  #                         alpha=0.3))+
  #geom_sf(data = coast2, colour="black", fill=NA)+
  #geom_sf(data = gridcrop1, colour="black", fill=NA)+
  #geom_sf(data= suba1aa, fill=NA)+
  # geom_sf(aes(fill=ssmu481aa$GAR_Short_Label,
  #              alpha=0.3))+
  scale_fill_viridis_d(option = "F",
                       name="Strata")+
  #geom_sf_label(aes(label = strata$ID))+
  # labs(fill = "SSMU")+
  ylim(230000, 2220000)+
  xlim(-3095349 , -1858911)+
  # coord_sf(crs = 32610)+ #sistema de prpyecccion para campos completos
  coord_sf(crs = 6932)+
  scale_alpha(guide="none")+
  theme_bw()
ssmap
```

<img src="index_files/figure-html/unnamed-chunk-6-1.jpeg" style="display: block; margin: auto;" />
plot simple 


```r
ssmap <- ggplot()+
  geom_sf(data = strata, aes(fill=strata$ID, 
                           alpha=0.3))+
  # geom_sf(data = ssmu481aa, aes(fill=ssmu481aa$GAR_Short_Label, 
  #                         alpha=0.3))+
  #geom_sf(data = coast2, colour="black", fill=NA)+
  #geom_sf(data = gridcrop1, colour="black", fill=NA)+
  #geom_sf(data= suba1aa, fill=NA)+
  # geom_sf(aes(fill=ssmu481aa$GAR_Short_Label,
  #              alpha=0.3))+
  scale_fill_viridis_d(option = "F",
                       name="Strata")+
  #geom_sf_label(aes(label = strata$ID))+
  # labs(fill = "SSMU")+
  ylim(230000, 2220000)+
  xlim(-3095349 , -1858911)+
  # coord_sf(crs = 32610)+ #sistema de prpyecccion para campos completos
  coord_sf(crs = 6932)+
  scale_alpha(guide="none")+
  theme_bw()
ssmap
```

<img src="index_files/figure-html/unnamed-chunk-7-1.jpeg" style="display: block; margin: auto;" />


```r
# comoprobar si tengo datos duplicados
strata2 <- st_make_valid(strata)
dataenvi <- st_make_valid(gsiv)
```


# Conclusion

-   On the one hand, the models with mixed effects serve to verify the
    influence of the spatial component, in this case each cell y in
    which the data of the dependent variable (krill sizes) and the
    independent variable (environmental variables) were considered.

-   The influence of environmental variables on the sizes of the krill
    fishery is corroborated. The environmental variable with the
    greatest impact on krill sizes is Chlorophyll in negative terms. In
    other words, the more chlorophyll in the environment, the sizes
    decrease because there is greater recruitment due to the abundance
    of substrate for the krill population.

-   In a way, it is proof that the krill population structure is
    influenced not only by fishing pressure, but also by environmental
    conditions.

-   With these results, the environmental component is solidly
    incorporated into the krill stock assessment model in the Antarctic
    Peninsula, specifically in Subarea 48.1.

# References



