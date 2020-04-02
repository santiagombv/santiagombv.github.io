---
layout: post
title: Ajustando una curva de crecimiento exponencial
---

La pandemia de **Covid-19** (coronavirus) ha promovido una explosión de gráficas y análisis a los que muchos no estabamos acostumbrados. Ajustes de crecimiento exponencial, escala logarítmica y otras alternativas son ahora moneda corriente en la prensa y entre "divulgadores".   

Descubrimos la sorpresa como el Marajá del cuento, quien debía pagar un grano de arroz por la primer casilla de un tablero de ajedrez, 2 por la segunda, 4 por la tercera... hasta llegar a ![2^{64}](https://render.githubusercontent.com/render/math?math=2%5E%7B64%7D).   

Muchos reclaman que se dejen los modelos a quienes saben de ello, pero la sencillez de una exponencial está muy lejos de los modelados de un epidemiólogo, y es accesible a cualquiera (con cierto entrenamiento en **R** en este caso). Todo lo contrario a este reclamo, pienso que poner las herramientas en manos de aquellos que quieran trabajar es una obligación. Este es uno de los motivos por el que dicto cursos de estadística con software libre desde hace 11 años.   

Así que va este post para tres modelados básicos. Luego subiré alguno de comparación de curvas mediante Modelos Aditivos Generalizados.   

***

Una manera conveniente de representar el crecimiento exponencial es mediante la fórmula. (Nota: estas fórmulas se construyeron con [esta app](https://alexanderrodin.com/github-latex-markdown/?math=N(t)%20%3D%20N_0%20e%5E%7B%5Clambda%20t%7D))    

![N(t) = N_0 e^{\lambda t}](https://render.githubusercontent.com/render/math?math=N(t)%20%3D%20N_0%20e%5E%7B%5Clambda%20t%7D)

Donde ![N(t)](https://render.githubusercontent.com/render/math?math=N(t)) es la cantidad de determinado organismo (en este caso, personas infectadas con Covid-19) para un momento determinado del tiempo, llamado *t*;*e* es la base de los logaritmos naturales (aproximadamente el número 2,7182...); ![\lambda](https://render.githubusercontent.com/render/math?math=%5Clambda) es el parámetro de crecimiento (si es mayor a 0) o de decrecimiento (si es menor) y ![N_0](https://render.githubusercontent.com/render/math?math=N_0) es una constante que bpasicamente representa el número inicial de personas infectadas (cuando el tiempo es igual a 0, pensar que en ese caso ![e^{\lambda t} = 1](https://render.githubusercontent.com/render/math?math=e%5E%7B%5Clambda%20t%7D%20%3D%201)).   

Hice un [repositorio en Github](https://github.com/santiagombv/COVID19AR_GAMs) donde subí los datos que el Ministerio de Salud *nos comunica oralmente* en sus informes matutinos, o bien *coloca en un pdf* del cual hay que extraer los datos a mano, en sus informes vespertinos. Los links a los informes están en la misma base de datos.

```r
# uso Bitly para acortar el url, solo por cuestión de formato
dat <- read.csv("https://bit.ly/33Z7Qk2", header = TRUE)

```

### Ajuste de una función exponencial
```r
# 1- ELEGIR EL DÍA CERO
# el día cero puede ser el día del primer caso, el día del 
# primer caso de transmisión comunitaria, o el día que se 
# alcanza una masa considerada "crítica" (lo cual requiere 
# otra estimación). Aquí vamos a tomar por simplicidad el 
# día en que ya hay 10 casos y a partir de eso construimos 
# un set de datos recortado

dat2 <- subset(dat, dat$casos_acum >=10)

# 2- Agregar una columna, con el DÍA DE LA EPIDEMIA
# contando desde el día cero definido arriba (esta es fácil)

dat2$dia <- c(1:nrow(dat2))

# 3- AJUSTE DEL MODELO EXPONENCIAL
# notar que la función nls (non linear least squares)
# requiere construir una fórmula con parámetros
# y luego brindar una "educada conjetura" sobre el valor
# de estos parámetros.
# En este caso es sencillo, el número inicial de infectados 
# es 5 (lo fijamos arriba) y el parámetro de crecimiento debe
# ser un número positivo (ya que modelamos crecimiento),
# aunque no muy alto

m1 <- nls(casos_acum ~ N0 * exp(lambda * dia),
          data = dat2,  
          start=list(N0=5, lambda=0.1))
summary(m1)

# 4- La ecuación exponencial tiene la agradable propiedad de que
# podemos calcular el TIEMPO DE DUPLICACIÓN

TD <-  log(2)/coefficients(m1)[2]
TD # resultado: 4.646365 días

# 5- GRÁFICO. 
# por conveniencia, suelo crear un set de datos nuevo cada para
# hacer los gráficos con el paquete ggplot2 (pero es una maña mía)

dat3 <- dat2[, c("fecha", "dia", "casos_acum")]

# al cual le agrego una columna de datos con predicciones
# nls CARECE de métodos para calcular el intervalo de confianza
dat3$pred <- predict(m1, newdata = dat3)
g1  <- ggplot(dat3, aes(x = dia, y = pred)) + geom_line()
g1 <- g1 + geom_point(aes(x = dia, y = casos_acum)) + theme_bw() 
```
   
![g1](/images/post2020-04-03-g1.png)   
   
```r
# cambiar a la escala logarítmica es muy fácil
g1 + scale_y_log10()
```
   
![g1b](/images/post2020-04-03-g1b.png)   
   
OBSERVACIÓN: en un ajuste por mínimos cuadrados como el que hace nls, los últimos puntos tienen más peso, sencillamente porque son valores más grandes, y la curva trata de aproximarlos mejor para dejar el mínimo residuo posible. Este modelado nos arroja un tiempo de duplicación de 4.646365 días.    

### Ajuste de una función lineal (en el espacio logarítmico)   

```r
m2 <- lm(log(casos_acum) ~ dia, data = dat2)
summary(m2)

# esto nos da una estimación un poco menos sesgada de los parámetros
B <- m2$coefficients

B[2] # parámetro de crecimiento = 0.1991357 
log(2)/B[2] # tiempo de duplicación = 3.480777

# otra vez, podemos plotear
# para usar la escala real, vamos a exponenciar la predicción
# (esto hace más fácil luego alternar entre el gráfico lineal
# y logaritmico usando ggplot2)
dat3 <- dat2[, c("fecha", "dia", "casos_acum")]
dat3$pred <- exp(predict(m2, newdata = dat3))
g2  <- ggplot(dat3, aes(x = dia, y = pred)) + geom_line()
g2 <- g2 + geom_point(aes(x = dia, y = casos_acum)) + theme_bw() 
g2

```
   
![g2](/images/post2020-04-02-g2.png)   
   
```r
# en escala logarítmica
g2 + scale_y_log10()
```
   
![g2b](/images/post2020-04-02-g2b.png)   
   
Vemos aquí cómo el sesgo se empieza a corregir, aunque el ajuste no es lineal claramente. El tiempo de duplicación se estima ahora en 3.4808 días.   


### WORK IN PROGRESS








