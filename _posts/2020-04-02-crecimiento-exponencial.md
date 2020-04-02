---
layout: post
title: Ajustando una curva de crecimiento exponencial
---

La pandemia de **Covid-19** (coronavirus) ha promovido una explosión de gráficas y análisis a los que muchos no estabamos acostumbrados. Ajustes de crecimiento exponencial, escala logarítmica y otras alternativas son ahora moneda corriente en la prensa y entre "divulgadores".   

Descubrimos la sorpresa como el Marajá del cuento, quien debía pagar un grano de arroz por la primer casilla de un tablero de ajedrez, 2 por la segunda, 4 por la tercera... hasta llegar a ![2^{64}](https://render.githubusercontent.com/render/math?math=2%5E%7B64%7D).   

Muchos reclaman que se dejen los modelos a quienes saben de ello, pero la sencillez de una exponencial está muy lejos de los modelados de un epidemiólogo, y es accesible a cualquiera (con cierto entrenamiento en **R** en este caso). Todo lo contrario a este reclamo, pienso que poner las herramientas en manos de aquellos que quieran trabajar es una obligación. Este es uno de los motivos por el que dicto cursos de estadística con software libre desde hace 11 años.   

Así que va este post para tres modelados básicos, el último, mediante **Modelos Aditivos generalizados**. El objetivo es hacer notar como diferentes modelados pueden llevara a diferentes conclusiones. A diferencia de otros posibles modelos, **NO** pretendo realizar predicciones sobre la evolución de la epidemia en Argentina, sino describir su evolución hasta ahora.   

***

### Crecimiento exponencial   

Una manera conveniente de representar el crecimiento exponencial es mediante la fórmula. (Nota: estas fórmulas se construyeron con [esta app](https://alexanderrodin.com/github-latex-markdown/?math=N(t)%20%3D%20N_0%20e%5E%7B%5Clambda%20t%7D) de Alexander Rodin).    

![N(t) = N_0 e^{\lambda t}](https://render.githubusercontent.com/render/math?math=N(t)%20%3D%20N_0%20e%5E%7B%5Clambda%20t%7D)

Donde ![N(t)](https://render.githubusercontent.com/render/math?math=N(t)) es la cantidad de determinado organismo (en este caso, personas infectadas con Covid-19) para un momento determinado del tiempo, llamado *t*;*e* es la base de los logaritmos naturales (aproximadamente el número 2,7182...); ![\lambda](https://render.githubusercontent.com/render/math?math=%5Clambda) es el parámetro de crecimiento (si es mayor a 0) o de decrecimiento (si es menor) y ![N_0](https://render.githubusercontent.com/render/math?math=N_0) es una constante que bpasicamente representa el número inicial de personas infectadas (cuando el tiempo es igual a 0, pensar que en ese caso ![e^{\lambda t} = 1](https://render.githubusercontent.com/render/math?math=e%5E%7B%5Clambda%20t%7D%20%3D%201)).   

### Datos

Hice un [repositorio en Github](https://github.com/santiagombv/COVID19AR_GAMs) donde subí los datos que el Ministerio de Salud *nos comunica oralmente* en sus informes matutinos, o bien *coloca en un pdf* del cual hay que extraer los datos a mano, en sus informes vespertinos. Los links a los informes están en la misma base de datos. Los análisis están realizados con información actualizada al 1 de abril de 2020.

```r
# uso Bitly para acortar el url, solo por cuestión de formato
# leo directamente desde la base en github
# colClasses es necesario para que identifique la primer 
# columna como fecha. NA indica que lea el resto de las 
# columnas normalmente

dat <- read.csv("https://bit.ly/33Z7Qk2", header = TRUE, 
                colClasses=c("Date", rep(NA, 16)))

```

### Elegir el día cero   

El día cero puede ser el día del primer caso, el día del primer caso de transmisión comunitaria, o el día que se alcanza una masa considerada "crítica" de infectados (lo cual requiere otra estimación). Aquí vamos a tomar por simplicidad el día en que ya hay 10 casos. A partir de esa fecha construimos un set de datos recortado.   

```r
dat2 <- subset(dat, dat$casos_acum >=10)

# Agregar una columna, con el DÍA DE LA EPIDEMIA contando desde el día 0
dat2$dia <- c(1:nrow(dat2))
```

### Ajuste de una función exponencial

La vamos a ajustar via la función notar que la función *nls* (non linear least squares). Esta función requiere construir una fórmula con ciertos parámetros y luego brindar una "conjetura educada " sobre el valor de estos parámetros, básicamente para acelerar el proceso de estimación. En este caso es sencillo, el número inicial de infectados es 10 (lo fijamos arriba) y el parámetro de crecimiento debe ser un número positivo (ya que modelamos crecimiento), aunque no muy alto.   

```r
m1 <- nls(casos_acum ~ N0 * exp(lambda * dia),
          data = dat2,  
          start=list(N0=5, lambda=0.1))
summary(m1)

# La ecuación exponencial tiene la agradable propiedad de que
# podemos calcular el TIEMPO DE DUPLICACIÓN

TD <-  log(2)/coefficients(m1)[2]
TD # resultado: 4.646365 días

# Gráfico (por conveniencia, suelo crear un set de datos nuevo cada 
# para hacer los gráficos con el paquete ggplot2 (pero es una maña mía)

dat3 <- dat2[, c("fecha", "dia", "casos_acum")]

# a este data set le agrego una columna de datos con predicciones
# WARNING: nls CARECE de métodos para calcular el intervalo de confianza

dat3$pred <- predict(m1, newdata = dat3)

# plot

library(ggplot2)
g1 <- ggplot(dat3, aes(x = fecha, y = pred)) + geom_line(color = "red")
g1 <- g1 + geom_point(aes(x = fecha, y = casos_acum)) + theme_bw() 
g1 

```
   
![g1](/images/post2020-04-02-g1.png)   
   
```r
# cambiar a la escala logarítmica es muy fácil
g1 + scale_y_log10()

```
   
![g1b](/images/post2020-04-03-g1b.png)   
   
Conclusión del modelado vía *nls*: en un ajuste por mínimos cuadrados, los últimos puntos tienen más peso, sencillamente porque son valores más grandes, y la curva trata de aproximarlos mejor para dejar el mínimo residuo posible. Este modelado nos arroja un tiempo de duplicación de 4.646365 días.    

### Ajuste de una función lineal (en el espacio logarítmico)   

Para esto vamos a usar sencillamente la función *lm* (linear models), logaritmizando el número de casos.

```r
m2 <- lm(log(casos_acum) ~ dia, data = dat2)
summary(m2)

# esto nos da una estimación un poco menos sesgada de los parámetros
B <- m2$coefficients

# el N0 necesita ser exponenciado
exp(B[1]) # 11.20149 
# pero no así el parámetro de crecimiento y la tasa de duplicación
B[2] # 0.1991357
log(2)/B[2] # 3.480777

# otra vez, podemos plotear
# para usar la escala real, vamos a exponenciar la predicción
# (esto hace más fácil luego alternar entre el gráfico lineal
# y logaritmico usando ggplot2)
dat3 <- dat2[, c("fecha", "dia", "casos_acum")]
dat3$pred <- exp(predict(m2, newdata = dat3))
g2  <- ggplot(dat3, aes(x = fecha, y = pred)) + geom_line(color="blue")
g2 <- g2 + geom_point(aes(x = fecha, y = casos_acum)) + theme_bw() 
g2

```
   
![g2](/images/post2020-04-02-g2.png)   
   
```r
# en escala logarítmica
g2 + scale_y_log10()
```
   
![g2b](/images/post2020-04-02-g2b.png)   
   
Conclusión del modelado lineal: El sesgo se empieza a corregir, aunque el ajuste tampoco es lineal claramente. El tiempo de duplicación se estima ahora en 3.4808 días.   

### Modelos Aditivos Generalizados (GAM)   

En un GAM la fórmula general de un modelo lineal generalizado.   

![g(\mu_i) = \beta_0 + \beta_1X_{1i}+...+\beta_kX_{ki}](https://render.githubusercontent.com/render/math?math=g(%5Cmu_i)%20%3D%20%5Cbeta_0%20%2B%20%5Cbeta_1X_%7B1i%7D%2B...%2B%5Cbeta_kX_%7Bki%7D)   

Es reemplazada por   

![g(\mu_i) = \beta_0 + f_1X_{1i}+...+f_kX_{ki}](https://render.githubusercontent.com/render/math?math=g(%5Cmu_i)%20%3D%20%5Cbeta_0%20%2B%20f_1X_%7B1i%7D%2B...%2Bf_kX_%7Bki%7D)   

Donde la muy, muy pequeña diferencia es que los parámetros ![\beta](https://render.githubusercontent.com/render/math?math=%5Cbeta) han sido reemplazados por funciones ![f](https://render.githubusercontent.com/render/math?math=f), cuyo único requisito es ser continuas y suaves. Estas funciones se ajustan además localmente (no son tan influidas por datos extremos como los ejemplos anteriores) y su "suavidad óptima" es obtenida por validación cruzada. Todo lo anterior es bastante técnico, pero su ajuste en *R* es mu sencillo. Por otra parte no he visto modelados del desarrollo de epidemias con **GAM** pero, dado que el objetivo es describir y no predecir, me parecen una herramienta al alcance. Hay además un entusiasmo por calcular los valores de las curvas exponenciales que el uso de **GAM** enfría, lo que me gusta.

```r
# el ajuste se realiza via el paquete mgcv
# la s dentro de la fórmula, indica que la relación entre el día 
# y los casos es aproximada mediante un spline, una función suave
# y continua

library(mgcv)
f1 <- gam(log(casos_acum) ~ s(dia), data = dat2) 
summary(f1) # no veremos por ahora como interpretarlo

#  Gráfico
# Como antes, creo un "toy data" para graficar en ggplot2

pred <- predict(f1, se.fit = TRUE)
DAT <- data.frame(
  fecha = dat2$fecha,
  casos = dat2$casos_acum,
  dia = dat2$dia,
  pred = exp(pred$fit), # exponencio la predicción
  up = exp(pred$fit + pred$se.fit), # y sus errores estandard
  down =  exp(pred$fit - pred$se.fit) # y sus errores estandard
)

g3 <- ggplot(DAT, aes(fecha, pred)) + geom_line(color = "purple3") 
g3 <- g3 + geom_point(aes(fecha, casos)) + theme_bw()
g3 <- g3 + geom_line(aes(fecha, up), linetype = 3, color = "purple3") 
g3 <- g3 + geom_line(aes(fecha, down), linetype = 3, color = "purple3")   
g3 

```

![g3](/images/post2020-04-02-g3.png)   

```r
# lo mismo en escala lineal
g3 + scale_y_log10()

```
 
![g3](/images/post2020-04-02-g3b.png)   

Los **GAM** ofrecen mejores maneras de representar curvas que se apartan de la linealidad (y de comparar curvas con diferentes complejidades). La defunción de los parámetros deja a mucha gente con sudor frío, pero de todas maneras pordemos calcular la pendiente de la curva en cada punto del *spline*. "Fácil", diferenciando en un intervalo pequeño (tomado a partir de la ayuda de *predict.gam*).   

```r
# primero predigo en el intervalo dado por el numero de días
newx0 <- seq(min(1, max(dat2$dia),length.out = 100)
Y0 <- predict(f1, data.frame(dia =newx0), type ="lpmatrix")

# luego hago exactamente lo mismo, pero le sumo un valor pequeño
eps <- 1e-7 
newx1 <- newx0 + eps
Y1 <- predict(f1, data.frame(dia =newx1), type ="lpmatrix")

# entre estos dos se crea una pequeña "pendiente"...
Yp <- (Y1-Y0)/eps

# el spline está descripto por 10 parámetros, de los cuales el primero
# no sirve porque es el intercepto
# ahora si, la predicción en el espacio real
df <- Yp[,2:10]%*%coef(f1)[2:10]              
df.sd <- rowSums(Yp[,2:10]%*%f1$Vp[2:10,2:10]*Yp[,2:10])^.5 # errores (baratos)

# gráfico básico
layout(matrix(1:2, 1, 2))
plot(newx0, df, type="l", ylim = c(0, 0.3),
     lwd = 2, col = "green3",
     xlab = "días desde casos = 10",
     ylab = "parámetro de crecimiento")
lines(newx0,df+2*df.sd,lty=2)
lines(newx0,df-2*df.sd,lty=2)

plot(newx0, log(2)/df, type="l", ylim = c(2.5, 9), 
     lwd = 2, col = "green3",
     xlab = "días desde casos = 10",
     ylab = "tiempo de duplicación")
lines(newx0, log(2)/(df+2*df.sd),lty=2)
lines(newx0, log(2)/(df-2*df.sd),lty=2)
layout(1)
```

¿Cambió el comportamiento de la epidemia? Sí, pero no podemos predecir si seguirá de esta forma, ni fue ese el objetivo (vean los intervalos de confianza del tiempo de duplicación). ¿Podemos ver otras variables de la evolución de la epidemia? Sí, deberíamos. Desde el 25 de marzo se reporta el número de internados en terapia intensiva (terapia_acum en la [base de datos que subí a Github](https://github.com/santiagombv/COVID19AR_GAMs). Dado que existe un número finito de camas de este tipo, este número es el que determina la saturación del sistema de salud o no. También existen diferencias entre provincias que podemos examinar con **GAM** (tema del próximo post).












