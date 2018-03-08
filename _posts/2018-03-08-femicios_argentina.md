---
layout: post
title: Femicidios y homicidios agravados por el género (Argentina, 2012-2018)
---

Tomé la base de datos abiertos del [Ministerio de Justicia,](http://datos.gob.ar/dataset/registro-sistematizacion-seguimiento-femicidios-homicidios-agravados-por-genero). Claramente es incompleta y algunos datos están mal cargados.   
Según esta página se registraron "femicidios y homicidios agravados por el género desde el año 2012 a la fecha. La base se nutre de diversas fuentes: artículos de prensa escrita, denuncias policiales y judiciales, denuncias realizadas ante la Secretaría de Derechos Humanos."   

```r
library(maps)
library(mapdata)
library(animation)

# Uso Google shortener https://goo.gl/ para el url.
dat <- read.csv("https://goo.gl/JvNvBy", header = T) #

x <- as.character(dat$lugar_hecho)   
x[x == ""] <- NA # no se registra la provincia de algunos casos   
dat$lugar_hecho <- as.factor(x)   

dat$fecha_hecho <- as.Date(dat$fecha_hecho) # convertimos a formato fecha  
# algunas fechas están mal puestas   
dat$fecha_hecho[dat$fecha_hecho > as.Date("2018-03-08")] <- NA 

# comprobar que haya 24 niveles (pueden existir nombres mal escritos)
levels(dat$lugar_hecho)
```

Obtuve los centros geográficos (aproximados) de cada provincia. La base está disponible desde un repositorio de Github.   
Combino y ordeno las dos bases de datos.
```r
coo <- read.csv("https://goo.gl/T5DRsC", header = T)
lookup <- data.frame(lugar_hecho=levels(dat$lugar_hecho), 
                     prov.lat=coo$lat, prov.lon =coo$long)
dat2 <- merge(dat, lookup, by="lugar_hecho")
o <- order(dat2$fecha_hecho)
dat2 <- dat2[o,]
```

La animación en si misma, crea el GIF que acompaña el post. Se agrega un contador que muestra la fecha de cada caso. La creación del GIF puede demorar varios minutos.   

```r
ani.options(interval=.05)
# "ruido" alrededor del centro geográfico para evitar superposición.
dat2$xn <- rnorm(nrow(dat2), sd=0.5) 
dat2$yn <- rnorm(nrow(dat2), sd=0.5)

# GIF en si mismo (revisar el directorio de trabajo)
saveGIF({
  for(i in 1:nrow(dat2)){
    map("worldHires","Argentina", fill=TRUE)
    map("worldHires","Falkland Islands", fill=TRUE, add = T)
    points(dat2$prov.lon[1:i]+dat2$xn[1:i], 
           dat2$prov.lat[1:i]+dat2$yn[1:i], 
           pch=19, col=rgb(0.98, 0, 0.553, 0.4),
           cex = 2)
    text(-60.331376+1, -36.393665-10, 
         dat2$fecha_hecho[i], 
         col = "black", cex = 0.9)
  }
}) 
```
Esta base, incompleta, contiene a la fecha de hoy 986 femicidios y homicidios agravados por el género a lo largo de 2198 días, desde el 22 de enero de 2012 al 28 de enero de 2018. Uno cada 2.23 días.

![Image description](/images/femicidios2012-2018.gif)
