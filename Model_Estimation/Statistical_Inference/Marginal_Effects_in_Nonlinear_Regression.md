---
title: Marginal Effects in Nonlinear Regression
parent: Statistical Inference
grand_parent: Model Estimation
has_children: false
nav_order: 1
mathjax: true ## Switch to false if this page has no equations or other math rendering.
---

# Marginal Effects in Nonlinear Regression

In [linear regression]({{ "/Model_Estimation/OLS/simple_linear_regression.html" | relative_url }}), the effect of a predictor can be interpreted directly in terms of the outcome variable. For example, in the model $$Y = \beta_0 + \beta_1X + \varepsilon$$, a one-unit increase in $$X$$ is associated with a $$\beta$$-unit change in $$Y$$.

However, in nonlinear regression, this is no longer the case. For example, if $$Y$$ is binary and $$E(Y=1) = F(\beta_0 + \beta_1X)$$ is estimated using a [logit regression]({{ "/Model_Estimation/GLS/logit_model.html" | relative_url }}) as a generalized linear model, then a one-unit increase in $$X$$ is associated with a $$\beta$$-unit change in the *index function* which then passes through the logit link function to produce a change in $$E(Y=1)$$. Confusing!

Commonly, we would prefer to state our results in terms of changes in the actual dependent variable, which can be more intuitive. Marginal effects are one way of doing this. The *marginal effect* of $$X$$ on $$Y$$ in that logit regression is the relationship between a one-unit change in $$X$$ and the probability that $$Y=1$$. 

Marginal effects can be calculated for all sorts of nonlinear models. This page will discuss only logit and probit, but the same concepts (and, often, code, especially for other generalized linear models) work for other nonlinear models.

The marginal effect can be calculated by taking the derivative of the outcome variable with respect to the predictor of interest. This is how effects can be interpreted in general. Even in a linear model like $$Y = \beta_0 + \beta_1X + \varepsilon$$, we can see that $$\partial{Y}/\partial{X} = \beta_1$$, i.e. a one-unit change in $$X$$ is associated with a $$\beta_1$$-unit change in $$Y$$.

For both probit and logit models in the form $$E(Y=1) = F(\beta_0 + \beta_1X)$$, where $$F()$$ is the link function, the marginal effect can be calculated as

$$\frac{\partial{E(Y=1)}}{\partial X} = \beta_1F(\cdot)(1-F(\cdot))$$

Where $$F(\cdot) = F(\beta_0 + \beta_1X)$$ is the predicted probability that $$Y=1$$ given the predictors in the model. This means that the marginal effect is different for each observation, since  the predicted probability is different for each observation. Also, since $$F(\cdot)(1-F(\cdot))$$ is highest near $$F(\cdot) = .5$$, this means that the marginal effect will be highest for observations with predicted probabilities near .5, and lowest for observations with predicted probabilities near 0 or 1. This makes sense - if you're already predicted to have $$Y = 1$$ with .99 probability, there's not much more room for your probability to increase anyway - the marginal effect must be small!

The fact that the marginal effect is different for each individual also means that in order to present a *single* marginal effect, you must make a decision about how to combine everyone's effect. There are several common approaches:

- The **Marginal Effect of a Representative** selects a single representative set of predictors (and thus a single representative predicted probability) and calculates the marginal effect for that representative.
- The **Marginal Effect at the Mean** is the Marginal Effect of a Representative again, but this time the "representative" has the mean values of all of the predictors in the model.
- The **Average Marginal Effect** calculates the marginal effect for each individual separately, and then takes the mean of the marginal effects.

The average marginal effect is generally considered preferred (unless there is a particular representative of interest), since it accounts for correlations between the predictors, and creates an easily interpretable marginal effect.

For more detail see [The Effect](https://theeffectbook.net/ch-StatisticalAdjustment.html#nonlinear-regression), which this page draws from thoroughly.

## Keep in Mind

- Some software commands default to the marginal effect at the mean, while others default to average marginal effects. Be sure you know which one you're getting - read the documentation!
- The marginal effect is necessarily a simplification of the actual nonlinear regression model - you are throwing a little information out. Be aware of that.
- Marginal effect of a representative and marginal effect at the mean calculate marginal effects at particular values of the predictors. This does not calculate the marginal effect for subgroups with those levels of the predictors. Instead, it sets predictors to those values for all observations and calculates marginal effects as though those are the true values.

## Also Consider

- While the code on this page will generalize to a number of different nonlinear regression methods beyond just probit and logit, it won't cover all of them. If these don't work, look for marginal effects estimators that are custom-suited to the method you're using.

# Implementations

## Python

The **statsmodels** package contains methods for estimating some nonlinear models, and has built-in marginal effects methods for many of those models.

```python
import statsmodels.formula.api as sm
from causaldata import restaurant_inspections
df = restaurant_inspections.load_pandas().data

# sm.logit wants the dependent variable to be numeric
df['Weekend'] = 1*df['Weekend']

# Use sm.logit to run logit
m1 = sm.logit(formula = "Weekend ~ Year", data = df).fit()

# See the result
# m1.summary() would also work
Stargazer([m1])

# And get average marginal effects
m1.get_margeff().summary()

# Or marginal effects at the mean
m1.get_margeff(at = 'mean').summary()
```

## R

The **marginaleffects** package 
([link](https://vincentarelbundock.github.io/marginaleffects/index.html)) covers a wide variety of marginal effects methods in R.

```r
# Load packages and data
library(marginaleffects)
library(causaldata)
data(restaurant_inspections)

# Run a (silly) logit model
mod1 = glm(Weekend ~ Year, 
           data = restaurant_inspections,
           family = binomial) # Default link function for binomial family is "logit"

# Use `marginaleffects()` to get the average marginal effects (AMEs) for all our 
# predictors
mfx1 = marginaleffects(mod1)

# Use `summary()` to pretty-print these AMEs
summary(mfx1)

# Use the `newdata = datagrid()` constructor argument to pick a representative 
# observation
marginaleffects(mod1, newdata = datagrid(Year = 2005))

# An alternative to AME is the marginal effect at the mean (MEM).
# We can use the `newdata = "mean"` convenience argument to retrieve MEMs.
marginaleffects(mod1, newdata = "mean")
```

## Stata

Stata comes with the built-in `margins` command for marginal effects.

```stata
* Get data from the causaldata package (ssc install causaldata)
causaldata restaurant_inspections.dta, use clear download

* Use the logit command to regress
logit weekend year

* and the margins command to get the average marginal effect for all predictors
margins, dydx(*)

* Use the at() specification to pick a representative observation
margins, dydx(*) at(year = 2005)

* Or use atmeans to use the mean of all predictors (for year... 2010.337? Marginal effect at the means is weird)
margins, dydx(*) atmeans
```
