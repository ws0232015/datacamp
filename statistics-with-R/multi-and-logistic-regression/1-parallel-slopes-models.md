# Parallel Slopes Model

Definition：
`lm(y ~ x + z, data = test)`, x: numerical, y: categories

## Visualizing parallel slopes models

```r
library(broom)
library(ggplot2)

mod <- lm(totalPr ~ wheels + cond, data = mario_kart)

data_space <- ggplot(augment(mod), aes(x = wheels, y = totalPr, color = cond)) + 
  geom_point()
  
# single call to geom_line()
data_space + 
  geom_line(aes(y = .fitted))
```
## Interpreting parallel slopes coefficients

`lm(hwy ~ displ + factor(year), data = mpg)`

* Intercept
* slope

## Three ways to describe a model

### Mathematical

* Equation: $y = \beta_0 + \beta_1 x_1 + \beta_2 x_2 + \epsilon$
* Residuals: $\epsilon \sim N(0, \sigma_\epsilon)$
* Coefficients: $\beta_0, \beta_1, \beta_2$

### Geometric

ggplot2

### Syntactic

```r
lm(hwy ~ displ + factor(year), data = mpg)
#> Call:
#> lm(formula = hwy ~ displ + factor(year), data = mpg)
#>
#> Coefficients:
#> (Intercept)      displ       factor(year)2008
#>      35.276      -3.611          1.402
```

