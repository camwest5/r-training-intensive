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

If you've got two numeric variables, you might want to examine covariance and correlation. These indicate how strongly the variables are linearly related. We'll need to use the `players$age` variable as well.

The covariance between "height_cm" and "age" is

```R
cov(players$height_cm, players$age)
```

Similarly, we can find the Pearson correlation coefficient between two columns. 

```R
cor(players$height_cm, players$age)
```

You can also specify "kendall" or "spearman" for their respective correlation coefficients

```R
cor(players$height_cm, players$age, method = "kendall")
cor(players$height_cm, players$age, method = "spearman")
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
lm("height_cm ~ age", players)
```

If we store this as a variable, we can then produce a summary of the results

```R
model <- lm("height_cm ~ age", players)
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
ggplot(players, aes(x = height_cm, y = age)) 
    + geom_point()
```

Then, you'll need to plot the regression as a line. For reference,

$$ y = \text{slope}\times x + \text{intercept}$$

So
```R
b0 <- model$coefficients[1]
b1 <- model$coefficients[2]

ggplot(players, aes(x = age, y = height_cm)) + 
    geom_point() + 
    geom_abline(intercept = b0, slope = b1)
```

### $t$-tests

We can also perform $t$-tests. Typically, these are performed to examine the statistical signficance of a difference between two samples' means. Let's examine whether that earlier groupby result for is accurate for heights, specifically, **are goalkeepers taller than non-goalkeepers?**

Let's start by creating a new column with the values
 
| | | 
| --- | --- |
| `FALSE` | Non-goalkeeper | 
| `TRUE` | Goalkeeper | 
 

```R
players <- players %>% 
  mutate(gk = positions == "Goalkeeper")
```

The $t$-test's goal is to check whether $\text{height_cm}$ depends on $\text{gk}$, so the formula is $\text{height\_cm}\sim\text{gk}$. This is given to the `t.test` function:

```R
t.test(height_cm ~ gk, data = players)
```

Yielding a p-value of $p<2.2\times10^{-16}$, indicating that the null-hypothesis (*heights are the same*) is extremely unlikely.

To visualise this result, it might be helpful to produce a histogram of the heights

```R
ggplot(players, 
       aes(x = height_cm, fill = gk)) + 
  geom_histogram(bins = 24)
```

### ANOVAs

What about the means of the other three? We could use an ANOVA to examine them. We use the `aov()` function for this. 

Let's start by making a new dataset without goalkeepers

```R
no_gk <- players %>% filter(gk == FALSE)
```

Next, we save the analysis of variance results

```R
res_aov <- aov(height_cm ~ positions, data = no_gk)
```

And examine them with `summary()`

```R
summary(res_aov)
```

Even without goalkeepers included, it looks like their positions are not all independent of height.

### $\chi^2$ tests

$χ^2$ tests are useful for examining the relationship of categorical variables by comparing the frequencies of each. Often, you'd use this if you can make a contingency table.

We only have one useful categorical variable here, "positions" (the others have too many unique values), so we'll need to create another. Let's see if there's a relationship between players' positions and names with the letter "a".

Make a binary column for players with the letter "a" in their names. To do this, we need to apply a string method to *all* the columns in the dataframe as follows

```R
players <- players %>%
  mutate(a_in_name = grepl("a", name))
```

> The `grepl` function perform pattern matching: it checks if the pattern `"a"` is inside the values in `name`.

Let's cross tabulate positions with this new column

```R
table(players$positions, players$a_in_name)
```

The $χ^2$ test's job is to examine whether players' positions depend on the presence of "a" in their name. To evaluate it we need to send the contingency table in:

```R
chisq.test(table(players$positions, players$a_in_name))
```

As expected, there is no signifcant relationship. A simple bar plot can help us here

```R
ggplot(players,
       aes(x = positions, fill = a_in_name)) + 
  geom_bar()
```

If we use the `position = "fill"` parameter to `geom_bar`, we'll see each as a proportion

```R
ggplot(players,
       aes(x = positions, fill = a_in_name)) + 
  geom_bar(position = "fill")
```

It looks as though the proportions are much the same.

### Generalised linear models

We'll finish by looking at Generalised Linear Models. The distributions they include are

* Binomial
* Poisson
* Gaussian (Normal)
* Gamma
* Inverse Gaussian
* A few *quasi* options

We'll use the *binomial* option to create logistic regressions.

Logistic regressions examine the distribution of binary data. For us, we can compare the heights of **goalkeepers vs non-goalkeepers** again. 

Now, we can model this column with height. We'll do the same as our $t$-test case, but this time we need to specify that `family = binomial` to ensure we'll get a logistic:


```python
res_logistic <- glm(gk ~ height_cm, family = binomial, data = players)
```

We can take a look at the results with

```python
summary(res_logistic)
```

Let's have a look at the summary:

```python
res.summary()
```

And we can then visualise it with `ggplot2`. We need to make *another* variable, because we need to replace `TRUE` $\rightarrow$ `1` and `FALSE` $\rightarrow$ `0` for the plot.

```R
players <- players %>% mutate(gk_numeric = as.numeric(gk))
```

Now we can plot the logistic regression. The fitted values (on the $y$-axis) are stored in `res_logistic$fitted.values`, but there are no provided $x$-values - these come from the `players` dataset. Use `geom_point()` for the data and `geom_line()` for the fit:

```python
ggplot(players, aes(x = height_cm, y = gk_numeric)) + 
  geom_point() + 
  geom_line(aes(y = res_logistic$fitted.values))
```