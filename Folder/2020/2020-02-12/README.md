
<!-- README.md is generated from README.Rmd. Please edit that file -->

``` r
#Bibliotecas necesarias
library(tidyverse)
library(lubridate)
library("rnaturalearth")
library("rnaturalearthdata")
library(gganimate)
library(ggthemes)
```

``` r
#Lectura de Datos
confirmados <- readr::read_csv("https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/archived_data/time_series/time_series_2019-ncov-Confirmed.csv")
muertes <- readr::read_csv("https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/archived_data/time_series/time_series_2019-ncov-Deaths.csv")
recuperados <- readr::read_csv("https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/archived_data/time_series/time_series_2019-ncov-Recovered.csv")
nCoV <- list(confirmados = confirmados,
           muertes = muertes,
           recuperados = recuperados)
save(nCoV, file = "ncov.Rdata")
```

``` r
load("ncov.Rdata")
l_confirmados <- nCoV$confirmados %>% pivot_longer(-c(`Province/State`,`Country/Region`,Lat,Long),names_to = "Fecha", values_to = "Conteo") %>% 
  filter(!is.na(Conteo)) %>% separate(col = Fecha,into = c("Fecha","Hora"), sep =" ") %>% 
  group_by_at(1:5)%>% 
  summarise(confirmados = max(Conteo)) %>% 
  mutate(Fecha=as.Date(Fecha,format = "%m/%d/%y")) %>% 
  ungroup() %>% 
  group_by_at(c(1,2,3,4,6)) %>% 
  summarise(Fecha = min(Fecha))
  
l_muertes <-  nCoV$muertes %>% pivot_longer(-c(`Province/State`,`Country/Region`,Lat,Long),names_to = "Fecha", values_to = "Conteo") %>% 
  filter(!is.na(Conteo)) %>% separate(col = Fecha,into = c("Fecha","Hora"), sep =" ") %>% 
  group_by_at(1:5)%>% 
  summarise(muertes = max(Conteo)) %>% 
  mutate(Fecha=as.Date(Fecha,format = "%m/%d/%y")) %>% 
  ungroup() %>% 
  group_by_at(c(1,2,3,4,6)) %>% 
  summarise(Fecha = min(Fecha))
  
l_recuperados <-  nCoV$recuperados %>% pivot_longer(-c(`Province/State`,`Country/Region`,Lat,Long),names_to = "Fecha", values_to = "Conteo") %>% 
  filter(!is.na(Conteo)) %>% separate(col = Fecha,into = c("Fecha","Hora"), sep =" ") %>% 
  group_by_at(1:5)%>% 
  summarise(recuperados = max(Conteo)) %>% 
  mutate(Fecha=as.Date(Fecha,format = "%m/%d/%y")) %>% 
  ungroup() %>% 
  group_by_at(c(1,2,3,4,6)) %>% 
  summarise(Fecha = min(Fecha))
```

``` r
full_ncov <- l_confirmados %>%   
  full_join(l_muertes) %>% 
  full_join(l_recuperados)
```

``` r
world <- ne_states( returnclass = "sf")
g <- ggplot(l_confirmados) +
  geom_sf(data = world,fill="grey15", color="grey20") + 
  theme_map()+
  labs(title = "Coronavirus ",
       caption = "Vizualization by @DuvanNievesRui1 | Data: 'Casos Nuevo Coronavirus' • Universidad John Hopkins")+
  stat_density2d(aes(x=Long,y=Lat, fill=..level..), geom = "polygon")+
  scale_fill_viridis_c(direction = -1, alpha = .1, option = "C")+
  theme(panel.background = element_rect(fill="grey10",color = "grey10"),
        plot.background = element_rect(fill="grey10"),
        plot.title = element_text(size=28, color="grey76",hjust = .5),
        plot.subtitle  = element_text(size=20, color="grey76", hjust = .5),
        plot.caption = element_text(size = 14,color = "grey76", hjust = .99),
        legend.text = element_blank(),
        legend.title = element_blank(),
        legend.background = element_rect(fill = "grey10"),
        legend.key.size = unit(.5,"cm"))+
  transition_time(Fecha) 
animate(g, renderer = gifski_renderer(),height = 500, width = 1000,fps = 5)
```

![](README_files/figure-gfm/unnamed-chunk-5-1.gif)<!-- -->

``` r
g1 <- ggplot(full_ncov) +
  geom_sf(data = world,fill="grey15", color="grey20") + 
  theme_map()+
  geom_point(data= .%>% filter(!is.na(confirmados)),aes(x=Long,y=Lat, alpha=confirmados), color = "#E7B800", shape = 21, size=3)+
  geom_point(data= .%>% filter(!is.na(muertes)),aes(x=Long,y=Lat, alpha=muertes), color = "#FC4E07", shape = 23, size=7, fill="#FC4E07")+
  geom_point(data= .%>% filter(!is.na(recuperados)),aes(x=Long,y=Lat, alpha=recuperados), color ="#8FD744", shape = 22, size=1, fill="#8FD744")+
  scale_alpha(range = c(.5,.7))+
  labs(title = "Casos Nuevos Coronavirus ",
       subtitle = "nCov",
       caption = "Vizualization by @DuvanNievesRui1 | Data: 'Casos Nuevo Coronavirus' • Universidad John Hopkins")+
  theme(legend.position = "none",
        panel.background = element_rect(fill="grey10", color = "grey10"),
        plot.background = element_rect(fill="grey10"),
        plot.title = element_text(size=28, color="grey76",hjust = .5),
        plot.subtitle  = element_text(size=20, color="grey76", hjust = .5),
        plot.caption = element_text(size = 14,color = "grey76", hjust = .99),
        legend.text = element_text(family = "Roboto Mono",
                                   size = 10,
                                   colour = "grey76"),
        legend.title = element_text(family = "Roboto Mono",
                                   size = 14,
                                   colour = "grey76"),
        legend.background = element_rect(fill = "grey10"),
        legend.key.size = unit(1.5,"cm"))+
  transition_time(Fecha) 
 
g1 
animate(g1, renderer = gifski_renderer(),height = 500, width = 1000,fps = 5)
```

![](README_files/figure-gfm/unnamed-chunk-6-1.gif)<!-- -->
