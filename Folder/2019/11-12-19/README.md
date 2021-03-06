
<!-- README.md is generated from README.Rmd. Please edit that file -->

``` r
#Bibliotecas necesarias
library(tidyverse)
library(pinochet)
library(presentes)
library(ggmap)
library(ggpointdensity)
library(ggthemes)
library(paletteer)
library(gganimate)
library(sf)
```

``` r
#Mapas base
chile_map <- get_map(location=c(left = -75.3, bottom = -55, right = -65, top = -19), zoom=6, source = "stamen", maptype="terrain-background",crop =F,force = T)
argentina <- st_read("https://github.com/Demzayat/varios/raw/master/provinciasND.geojson")

save(chile_map,file = "chile-map.Rdata")
save(argentina,file = "argentina.Rdata")
```

``` r
#Datos
pinochet <- pinochet
centros_clandestinos_detencion <- centros_clandestinos_detencion
parque_de_la_memoria <- parque_de_la_memoria
victimas_accionar_represivo_ilegal <- victimas_accionar_represivo_ilegal
victimas_accionar_represivo_ilegal_sin_denuncia_formal <- victimas_accionar_represivo_ilegal_sin_denuncia_formal
```

# Pinochet

``` r
load("chile-map.Rdata")

violencia_chile <- pinochet %>% 
  select(violence, latitude_1, longitude_1) %>%
  filter(!is.na(violence), !is.na(latitude_1), !is.na(longitude_1)) %>% 
  rename(lat = latitude_1, lon =longitude_1) %>% 
  mutate(violencia = if_else(violence == "Disappearance","Desaparición",
                             if_else(violence =="Killed","Asesinado",
                                     if_else(violence=="Disappearance, Information of Death",paste("Desaparición","Información de muerte",sep = "\n"), 
                                             if_else(violence=="Suicide","Suicidio","Sin resolver")))))

g <- ggmap(chile_map)  +
  facet_grid(~violencia) +
  geom_pointdensity(data = violencia_chile, aes(x=lon,y=lat),size=5, alpha=.4) +
  scale_color_paletteer_c("ggthemes::Red-Green-Gold Diverging",direction = -1)+
  labs(title = "Abusos Derechos Humanos",
       subtitle = "Régimen de Pinochet",
       color = "Densidad",
       caption = "Vizualization by @DuvanNievesRui1 | Data: 'Pinochet' • Package Pinochet")+
  theme_map()+
  theme(legend.background = element_rect(fill = "transparent"),
        legend.position = "top",
        legend.key.size = unit(1,'cm'),
        legend.justification = "center",
        plot.margin = unit(rep(.5,4),'cm'),
        panel.background = element_rect(fill="grey10",color = "grey10"),
        plot.background = element_rect(fill="grey10"),
        strip.background = element_rect(fil="grey10", color="grey10"),
        plot.title = element_text(size=20, color="grey76",hjust = .5),
        plot.subtitle = element_text(size=15, color="grey76",hjust = .5),
        plot.caption = element_text(size = 12,color = "grey76", hjust = .98),
        strip.text.x =element_text(family = "Roboto Mono",
                                   size = 9,
                                   colour = "grey76"),
        legend.title = element_text(family = "Roboto Mono",
                                   size = 12,
                                   colour = "grey76"),
        legend.text = element_text(family = "Roboto Mono",
                                   size = 8,
                                   colour = "grey76")) +
  transition_layers(layer_length = .001,transition_length = 2,layer_order = c(2,4,3,1)) +
  shadow_mark()+
  enter_fade() + enter_grow() +
  exit_fade() + exit_shrink()

animate(g, renderer = gifski_renderer(),height = 750, width =750,fps = 20)  
```

<img src="README_files/figure-gfm/unnamed-chunk-5-1.gif" style="display: block; margin: auto;" />

``` r
load("argentina.Rdata")
#/U25BC

ggplot()+
  geom_sf(data = argentina, aes(fill=area_km2))+
  geom_point(data = centros_clandestinos_detencion %>%  filter(!is.na(lon)), aes(x=lon, y=lat),shape="\U21E3", color ="#770000FF",size=10, alpha=.5)+
  coord_sf(xlim = c(-52, -74), ylim = c(-20, -56), expand = F) +
  scale_fill_paletteer_c("grDevices::Greens 3")+
  theme_map()+
  theme(legend.position = "none")
```

<img src="README_files/figure-gfm/unnamed-chunk-6-1.png" style="display: block; margin: auto;" />
