---
title: R Training Intensive
format: 
  html:
    include-in-header:
      - text: <meta name="google-site-verification" content="lH7jPI_Tg64yiGhs_IV-_zClImxRtEK7aGXhAEl68g0" />
---

## Overview
Welcome to our three-day R training intensive! By Thursday afternoon, you'll have learnt the R skills to manipulate, visualise and present data. We'll spend roughly half the time learning content, and half the time working on a project in groups.

As we set up, there's a few things to do, if you haven't already

1. [Install the software](https://rstudio.com/products/rstudio/)
2. Introduce yourself to your table
3. [Join our Teams channel](https://forms.office.com/Pages/ResponsePage.aspx?id=z3fjtrOdy0aRovrZYFuxXEzmI13TctBBiWhneXZv-1lUQ1MwQ0JUM0ZBS0hXV1NJSU04TEZEUFg2Si4u)
4. [Register your attendance]()

## R + RStudio

The [R programming language](https://cran.r-project.org/) is a language
used for calculations, statistics, visualisations and many more data
science tasks.

[RStudio](https://rstudio.com/products/rstudio/) is an open source
Integrated Development Environment (IDE) for R, which means it provides
many features on top of R to make it easier to write and run code.

R’s main strong points are:

- **Open Source**: you can install it anywhere and adapt it to your
  needs;
- **Reproducibility**: makes an analysis repeatable by detailing the
  process in a script;
- **Customisable**: being a programming language, you can create your
  own custom tools;
- **Large datasets**: it can handle very large datasets (certainly well
  beyond the row limitations of Excel, and even further using
  [HPCs](https://rcc.uq.edu.au/high-performance-computing) and [other
  tricks](https://rviews.rstudio.com/2019/07/17/3-big-data-strategies-for-r/));
- **Diverse ecosystem**: packages allow you to extend R for thousands of
  different analyses.

The learning curve will be steeper than point-and-click tools, but as
far as programming languages go, R is more user-friendly than others.

### Installation

For this course, you need to have both R and RStudio installed
([installation
instructions](https://github.com/uqlibrary/technology-training/blob/master/R/Installation.md#r--rstudio-installation-instructions)).

## R Projects

Let’s first create a new project:

- Click the “File” menu button (top left corner), then “New Project”
- Click “New Directory”
- Click “New Project”
- In “Directory name”, type the name of your project, for example
  “YYYY-MM-DD_rstudio-intro”
- Browse and select a folder where to locate your project (`~` is your
  home directory). For example, a folder called “r-projects”.
- Click the “Create Project” button

> R Projects make your work with R more straight forward, as they allow
> you to segregate your different projects in separate folders. You can
> create a .Rproj file in a new directory or an existing directory that
> already has R code and data. Everything then happens by default in
> this directory. The .Rproj file stores information about your project
> options, and allows you to go straight back to your work.