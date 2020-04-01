---
layout: post
title: Ajustando una curva de crecimiento exponencial
---

La pandemia de Covid-19 (coronavirus) ha promovido una explosión de gráficas y análisis a los que muchos no estabamos acostumbrados. Ajustes de crecimiento exponencial, escala logarítmica y otras alternativas son ahora moneda corriente en la prensa.   

Descubrimos la sorpresa como el Marajá del cuento, quien debía pagar un grano de arroz por la primer casilla de un tablero de ajedrez, 2 por la segunda, 4 por la tercera... hasta llegar a $2^{64}$.   

Muchos reclaman que se dejen los modelos a quienes saben de ello, pero la sencillez de una exponencial está muy lejos de los modelados de un epidemiólogo, y es accesible a cualquiera (con cierto entrenamiento en **R** en este caso). Todo lo contrario a este reclamo, pienso que poner las herramientas en manos de aquellos que quieran trabajar es una obligación. No por nada hace 11 años que dicto cursos de estadística con software libre.   

Así que va este post para modelados básicos. Luego subiré alguno de comparación de curvas mediante Modelos Aditivos Generalizados.   

***

Una manera conveniente de representar el crecimiento exponencial es mediante la fórmula. (Nota: estas fórmulas se construyeron con [esta app](https://alexanderrodin.com/github-latex-markdown/?math=N(t)%20%3D%20N_0%20e%5E%7B%5Clambda%20t%7D))    

![N(t) = N_0 e^{\lambda t}](https://render.githubusercontent.com/render/math?math=N(t)%20%3D%20N_0%20e%5E%7B%5Clambda%20t%7D)

Donde $N(t)$ es la cantidad de determinado organismo (en este caso, personas infectadas con Covid-19) para un momento determinado del tiempo, llamado $t$;$e$ es la base de los logaritmos naturales (aproximadamente el número 2,7182...); $\lambda$ es el parámetro de crecimiento (si es mayor a 0) o de decrecimiento (si es menor) y $N_0$ es una constante que bpasicamente representa el número inicial de personas infectadas (cuando el tiempo es igual a 0, pensar que en ese caso $e^{\lambda t} = 0$).   

Hice un [repositorio en Github](https://github.com/santiagombv/COVID19AR_GAMs) donde subí los datos que el Ministerio de Salud *nos comunica oralmente* en sus informes matutinos, o bien *coloca en un pdf* del cual hay que extraer los datos a mano, en sus informes vespertinos. Los links a los informes están en la misma base de datos.

```r
# uso Bitly para acortar el url, solo por cuestión de formato
dat <- read.csv("https://bit.ly/33Z7Qk2", header = TRUE)

```

### WORK IN PROGRESS








