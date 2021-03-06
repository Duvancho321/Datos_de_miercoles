
<!-- README.md is generated from README.Rmd. Please edit that file -->

``` r
#Bibliotecas necesarias
library(readr)
library(tidyr)
library(dplyr)
library(ggplot2)
library(gganimate)
library(ggsci)
```

``` r
#Lectura de Datos
#empleo_genero <- readr::read_csv("https://raw.githubusercontent.com/cienciadedatos/datos-de-miercoles/master/datos/2019/2019-10-16/empleo_genero.csv")
#save(empleo_genero,file = "Empleo.Rdata")
load("Empleo.Rdata")
```

``` r
#Seleccion de datos
Datos <- empleo_genero %>%
  #Seleccion de las variables para cambiar a formato largo
  pivot_longer(-c(variable,codigo_pais_region,pais_region),
               names_to ="año",
               values_to = "porcentaje") %>% 
  #Renombrando el codigo del pais
  rename(codigo=codigo_pais_region) %>% 
  #Cambiando la variable año de caracter a numerico y codigo a factor
  mutate(año = as.numeric(año),
         codigo =as.factor(codigo) ) %>% 
  #Selecionando variable de interes "Desempleo"
  filter(variable == "desempleo_mujeres" |  variable == "desempleo_hombres") %>% 
  #Eliminando valores faltantes
  filter(!is.na(porcentaje)) %>% 
  #quitando la variable pais region
  select(-pais_region) %>%  
  #Filtrando paises en frontera con colobia
  filter(codigo == "COL" | codigo =="ECU" | codigo =="PER" | codigo =="VEN" | codigo =="BRA" | codigo =="PAN")
```

``` r
#Figura
gplot <- ggplot(Datos,aes(x=año,y=porcentaje,color=variable)) + 
  geom_line()+
  facet_wrap(~codigo,scales = "free")+
  scale_color_jco(name="Desempleo",
                      breaks = c("desempleo_hombres","desempleo_mujeres"),
                      labels=c("Hombres","Muejeres"))+
  theme_dark()+
  geom_tile(color="orange")+
  theme(text = element_text(size=10),
        legend.text = element_text( size = 12),
        legend.title = element_text(size=14),
        legend.position = "top",
        panel.grid.major = element_blank(),
        panel.grid.minor = element_blank(),
        panel.background = element_rect(fill = 'black'),
        legend.background = element_rect(fill = "white", color = NA),
        legend.key = element_rect(fill = "white"))+
  labs(x="Año",y="Porcentaje")+
  transition_reveal(año)

animate(gplot, renderer = gifski_renderer())
```

<img src="README_files/figure-gfm/unnamed-chunk-4-1.gif" style="display: block; margin: auto;" />
