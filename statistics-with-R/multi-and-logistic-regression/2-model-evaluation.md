# Model fit,residuals and prediction

## Model Fit

* Recall: $R^2 = 1 - \frac{SSE}{SST}$ (measures the percentage of the variability in the response variable that is explained by the model)
* SSE get smaller $\Rightarrow R^2$ increases...

* $R^2_{adj} = 1- \frac{SSE}{SST} \times \frac{n-1}{n-p-1}$ (The adjusted $R^2_{adj}$ includes a term that penalizes a model for each additional explanatory variable,where `p` is the number of explanatory variables)

## Fitted values

```r
# returns a vector
predict(mod)

# returns a data.frame
broom::augment(mod)
```

## Predictions

```r
new_obs <- data.frame(displ = 1.8, year = 2008)

predict(mode, newdata = new_obs)

augment(mod, newdata = new_obs)
```

# Understanding interaction

## Adding interaction terms

`lm(hwy ~ displ + factor(year) + displ:factor(year), data = mpg)`

## Simpson's Paradox

```r
SAT_wbin <- SAT %>%
  mutate(sat_bin = cut(sat_pct, 3))
  
mod <- lm(formula = total ~ salary + sat_bin, data = SAT_wbin)

ggplot(data = SAT_wbin, aes(x = salary, y = total, color = sat_bin)) + 
  geom_point() + 
  geom_line(data = broom::augment(mod), aes(y = .fitted))
```