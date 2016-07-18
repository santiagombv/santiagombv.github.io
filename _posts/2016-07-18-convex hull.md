---
published: false
---
## How to: compare convex hulls

A convex hull of a set of points is the smallest space of area that enclosed all of these points. If the points are populations or species in a multivariate morphological space, the convex hull is a meassure of disparity, i.e. how much is occupied the morphological space.
In [Maubecin et al. 2016,](http://onlinelibrary.wiley.com/doi/10.1111/jeb.12889/abstract?userIsAuthenticated=false&deniedAccessCustomisedMessage=) we tested whether floral morphological disparity was significantly larger in recently colonized than in refugium populations of _Calceolaria polyrhiza_. Here I provide the routine to repeat the test.

```
# The function dispar.hull estimates the convex hull of two sets of points, using principal
# components to reduce dimensionality.

# ID is a factor with two levels (identifiers).
# data is a data frame or matrix with at least two numeric variables.
# k is the number of principal compontents to be retained. By defoult k = 2, thus the function 
# estimates the convex hull areas. If k equals the number of variables in data, all the 
# information in the data is used to estimate the convex hull volumes.
# obs when TRUE (default) the function estimates the observed convex hulls. When FALSE the
# function randomly reshuffled the identifiers.

require(geometry)
dispar.hull <- function(ID, data, k = 2, obs = TRUE){
  if(obs == TRUE) ID <- ID else ID <- sample(ID)
  id <- unique(ID)
  PC <- prcomp(data)$x[,1:k]
  D1 <- subset(PC, ID == id[1])
  ch1 <- convhulln(D1)
  PA1 <- polyarea(D1[ch1[,1],1], D1[ch1[,2],2]); PA1
  D2 <- subset(PC, ID == id[2])
  ch2 <- convhulln(D2)
  PA2 <- polyarea(D2[ch2[,1],1], D2[ch2[,2],2]) 
  res <- c(PA1, PA2)
  return(res)
}

# Example
# Simulate some data
set.seed(123)
A <- matrix(rnorm(250, 0, 1), 50, 5)
B <- matrix(rnorm(250, 0.2, 0.35), 50, 5)
M <- as.data.frame(rbind(A, B))
M$type <- c(rep("a", 50), rep("b", 50))

# There is a huge difference in convex hull areas
library(vegan)
PC <- prcomp(M[, 1:5])
plot(PC$x[1:50, 1], PC$x[1:50,2], xlab = "PC1", ylab = "PC2")
points(PC$x[51:100, 1], PC$x[51:100,2], pch = 19)
ordihull(PC, groups = M$type)

# Area of the convex hulls
hull.obs <- dispar.hull(ID = M$type, data=M[, 1:5], obs = TRUE, k = 2)
hull.obs

# Randomization and significance of the difference between convex hulls
pseu.hull <- t(replicate(1000, dispar.hull(ID = M$type, data=M[, 1:5], obs = FALSE, k = 2)))
D <- pseu.hull[, 1] - pseu.hull[, 2]
hist(D); abline(v = hull.obs[1]- hull.obs[2])
P <- length(D[D > hull.obs[1]-hull.obs[2]])/1000
P
```
