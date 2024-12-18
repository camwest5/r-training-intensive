---
title: Statistics
--- 

This session is aimed as an overview of how to perform some statistical modelling with R. **It is an R workshop, not a statistics workshop** - if you'd like to better understand the statistical models, or need help deciding what's best for you, please consult a statistics resource or contact a statistician.

In this session, we'll cover

- Descriptive statistics
  - Measures of central tendancy
  - Measures of variability
  - Measures of correlation
  
- Inferential statistics
  - Linear regressions
  - Calculating confidence intervals
  - T-tests
  - $\chi^2$ test
  - ANOVAs

We'll be working from our "Players2024" dataset. To bring it in and clean it up,

```R
library(dplyr)
players <- read.csv("data_sources/Players2024.csv")
players <- players %>% filter(positions != "Missing", height_cm > 100)
```

## Descriptive Statistics

We'll start with sample size. To calculate the number of non-empty observations in a column, say the numeric variable `players$height_cm`, we use the `length()` function

```R
length(players$height_cm)
```

We can compute measures of central tendancy similarly. The average value is given by

```R
mean(players$height_cm)
```

and the median by

```R
median(players$height_cm)
```

### Measures of variance

We can also compute measures of variance. The minimum and maximum are as expected

```R
min(players$height_cm)
max(players$height_cm)
```

The function `range()` yields both

```R
range(players$height_cm)
```

So the actual range, i.e. the difference, is

```R
diff(range(players$height_cm))
```

Quartiles are given by `quantile()` and the inter-quartile range (IQR)  by `IQR()`:

```R
quantile(players$height_cm)
IQR(players$height_cm)
```

A column's standard deviation and variance are given by

```R
sd(players$height_cm)
var(players$height_cm)
```

All together, you can see a nice statistical summary with

```R
summary(players$height_cm)
```

### Measures of correlation

If you've got two numeric variables, you might want to examine covariance and correlation. These indicate how strongly the variables are linearly related. We'll need to use the `players$Age` variable as well.

The covariance between "height_cm" and "Age" is

```R
cov(players$height_cm, players$Age)
```

Similarly, we can find the Pearson correlation coefficient between two columns. 

```R
cor(players$height_cm, players$Age)
```

You can also specify "kendall" or "spearman" for their respective correlation coefficients

```R
cor(players$height_cm, players$Age, method = "kendall")
cor(players$height_cm, players$Age, method = "spearman")
```

### Reminder about groupbys

Before we move to inferential statistics, it's worth reiterating the power of groupbys discussed in the second workshop.

To group by a specific variable, like "positions", we use 

```R
players %>% 
    group_by(positions)
```

By applying our statistics to the `group_by` object, we'll apply them to *every* variable for *each* position.

```R
players %>% 
    group_by(positions) %>% 
    summarise(mean_height = mean(height_cm))
```

## Inferential Statistics

While descriptive statistics describes the data definitively, inferential statistics aim to produce models for extrapolating conlusions.

### Simple linear regressions

Least-squares regression for two sets of measurements can be performed with the function `lm`. Recall that linear regressions have the mathematical form

$$ Y = β_1 X + β_0 $$

and we use the regression tool to estimate the parameters $β_0\,,β_1$. We can equivalently say that $Y \sim X$, which is what R takes in

```R
lm("height_cm ~ Age", players)
```

If we store this as a variable, we can then produce a summary of the results

```R
model <- lm("height_cm ~ Age", players)
summary(model)
```

If you want to get specific parameters out, we can index with `$`:

```R
summary(model)$r.squared
```

That's a pretty shocking fit.

#### Plotting it

Naturally, you'd want to plot this. We'll need to use techniques from the visualisation session. Let's import **ggplot2**

```R
library(ggplot2)
```

Start by making a scatterplot of the data,

```R
ggplot(players, aes(x = height_cm, y = Age)) 
    + geom_point()
```

Then, you'll need to plot the regression as a line. For reference,

$$ y = \text{slope}\times x + \text{intercept}$$

So
```R
b0 <- model$coefficients[1]
b1 <- model$coefficients[2]

ggplot(players, aes(x = Age, y = height_cm)) + 
    geom_point() + 
    geom_abline(intercept = b0, slope = b1)
```

### $t$-tests

We can also perform $t$-tests with the `scipy.stats` module. Typically, this is performed to examine the statistical signficance of a difference between two samples' means. Let's examine whether that earlier groupby result for is accurate for heights, specifically, **are goalkeepers taller than non-goalkeepers?**

Let's start by separating the goalkeepers from the non-goalkeepers in two variables

```python
goalkeepers = df[df["positions"] == "Goalkeeper"]
non_goalkeepers = df[df["positions"] != "Goalkeeper"]
```

The $t$-test for the means of two independent samples is given by

```python
stats.ttest_ind(goalkeepers["height_cm"], non_goalkeepers["height_cm"])
```

Yielding a p-value of $8\times 10^{-247}\approx 0$, indicating that the null-hypothesis (*heights are the same*) is extremely unlikely.

### ANOVAs

What about the means of the other three? We could use an ANOVA to examine them. We use the `stats.f_oneway()` function for this. However, this requires us to send a list of samples in for each group, so we should separate the three positions. 

```python
defender = df[df["positions"] == "Defender"].height_cm
midfield = df[df["positions"] == "Midfield"].height_cm
attack = df[df["positions"] == "Attack"].height_cm
```

We can then perform the ANOVA on this list of samples

```python
stats.f_oneway(defender, midfield, attack)
```

With $p = 3\times10^{-84}$, it looks like their positions are not all independent of height.

### $\chi^2$ tests

$χ^2$ tests are useful for examining the relationship of categorical variables by comparing the frequencies of each. Often, you'd use this if you can make a contingency table.

We only have one useful categorical variable here, "positions" (the others have too many unique values), so we'll need to create another. Let's see if there's a relationship between players' positions and names with the letter "a".

Make a binary column for players with the letter "a" in their names. To do this, we need to apply a string method to *all* the columns in the dataframe as follows

```python
df["a_in_name"] = df["name"].str.contains("a")
```

Let's cross tabulate positions with this new column

```python
a_vs_pos = pd.crosstab(df["positions"],df["a_in_name"])
print(a_vs_pos)
```

The $χ^2$ test's job is to examine whether players' positions depend on the presence of "a" in their name. To evaluate it we need to send the contingency table in:

```python
stats.chi2_contingency(a_vs_pos)
```

### More complex modelling

If you need to do more advanced statistics, particularly if you need more regressions, you'll likely need to turn to a different package: `statsmodels`. It is particularly useful for **statistical modelling**.

We'll go through three examples

1. Simple linear regressions (like before)
2. Multiple linear regressions
3. Logistic regressions

What's nice about `statsmodels` is that it gives an R-like interface and summaries.

To start with, let's import the tools. We'll use the *formula* interface, which offers us an R-like way of creating models.

```python
import statsmodels.formula.api as smf
```

#### Simple linear regressions revisited

Let's perform the same linear regression as before, looking at the "Age" and "height variables". Our thinking is that players' heights dictate how long they can play, so we'll make $x = \text{height\_cm}$ and $y = \text{Age}$.

The first step is to make the set up the variables. We'll use the function `smf.ols()` for ordinary least squares. It takes in two imputs:

* The formula string, in the form `y ~ X1 + X2 ...`
* The data

We create the model and compute the fit

```python
mod = smf.ols("Age ~ height_cm", df)
res = mod.fit()
```

Done! Let's take a look at the results

```python
res.summary()
```

That's a lot nicer than with scipy. We can also make a plot by getting the model's $y$ values with `res.fittedvalues`

```python
sns.relplot(data = df, x = "height_cm", y = "Age")
sns.lineplot(x = df["Age"], y = res.fittedvalues, color = "black")
```

#### Generalised linear models

The `statsmodels` module has lots of advanced statistical models available. We'll take a look at one more: Generalised Linear Models. The distributions they include are

* Binomial
* Poisson
* Negative Binomial
* Gaussian (Normal)
* Gamma
* Inverse Gaussian
* Tweedie

We'll use the *binomial* option to create logistic regressions.

Logistic regressions examine the distribution of binary data. For us, we can compare the heights of **goalkeepers vs non-goalkeepers** again. Let's make a new column which is `1` for goalkeepers and `0` for non-goalkeepers:

```python
df["gk"] = (df["positions"] == "Goalkeeper")*1
```

> We have to multiply by 1 to turn `True` $\rightarrow$ `1` and `False` $\rightarrow$ `0`

Now, we can model this column with height. Specifically,

$$ \text{gk} \sim \text{height\_cm}$$

Start by making the model with the function `smf.glm()`. We need to specify the family of distributions; they all live in `sm.families`:

```python
mod = smf.glm("gk ~ height_cm", data = df, family = sm.families.Binomial())
```

Next, evaluate the results

```python
res = mod.fit()
```

Let's have a look at the summary:

```python
res.summary()
```

Finally, we can plot the result like before

```python
sns.relplot(data = df, x = "height_cm", y = "gk")
sns.lineplot(x = df["height_cm"], y = res.fittedvalues, color = "black")
```