
<!-- README.md is generated from README.Rmd. Please edit that file -->

# SDMtune <img src="man/figures/logo.svg" align="right" alt="" width="120" />

[![Travis-CI build
status](https://travis-ci.org/ConsBiol-unibern/SDMtune.svg?branch=master)](https://travis-ci.org/ConsBiol-unibern/SDMtune)
[![AppVeyor build
status](https://ci.appveyor.com/api/projects/status/github/sgvignali/SDMtune?branch=master&svg=true)](https://ci.appveyor.com/project/sgvignali/SDMtune)
[![Coverage
status](https://codecov.io/gh/ConsBiol-unibern/SDMtune/branch/master/graph/badge.svg)](https://codecov.io/github/ConsBiol-unibern/SDMtune?branch=master)
[![CRAN
Status](https://www.r-pkg.org/badges/version/SDMtune)](https://cran.r-project.org/package=SDMtune)
[![CRAN RStudio mirror
downloads](http://cranlogs.r-pkg.org/badges/grand-total/SDMtune)](http://www.r-pkg.org/pkg/SDMtune)
[![Contributor
Covenant](https://img.shields.io/badge/Contributor%20Covenant-v1.4%20adopted-ff69b4.svg)](.github/CODE_OF_CONDUCT.md)

**SDMtune** provides a user-friendly framework that enables the training
and the evaluation of species distribution models (SDMs). The package
implements functions for data driven variable selection and model tuning
and includes numerous utilities to display the results. All the
functions used to select variables or to tune model hyperparameters have
an interactive real-time chart displayed in the RStudio viewer pane
during their execution. SDMtune uses its own script to predict MaxEnt
models, resulting in much faster predictions for large datasets compared
to native predictions from the use of the Java software. This reduces
considerably the computation time when tuning the model using the AICc.
At the moment only the Maximum Entropy method is available using the
Java implementation through the “dismo” package and the R implementation
through the “maxnet” package.  
Visit the [package website](https://consbiol-unibern.github.io/SDMtune/)
and learn how to use **SDMtune** starting from the first article
[Prepare data for the
analysis](https://consbiol-unibern.github.io/SDMtune/articles/articles/prepare_data.html).

## Installation

You can install the latest development version from GitHub:

``` r
devtools::install_github("ConsBiol-unibern/SDMtune")
```

## Real-time charts

Real-time charts displaying the training and the validation metrics are
displayed in the RStudio viewer pane during the execution of the tuning
and variable selection functions.

<div style="text-align: center">

<img src="man/figures/realtime-chart.gif" alt="" />

</div>

## Speed test

Let’s see **SDMtune** in action. If the following code is not clear,
please check the articles in the
[website](https://consbiol-unibern.github.io/SDMtune/). Here we prepare
the data and we train a **Maxent** model using **SDMtune**:
<!-- The next code is not evaluated because MaxEnt jar file is bundled in the package and Travis will not execute it! -->
<!-- the plot is saved as an image in the man/figures forlder -->

``` r
# Acquire environmental variables
files <- list.files(path = paste(system.file(package = "dismo"), "/ex", sep = ""), pattern = "grd", full.names = TRUE)
predictors <- raster::stack(files)
# Prepare presence locations
p_coords <- condor[, 1:2]
# Prepare background locations
set.seed(25)
bg_coords <- dismo::randomPoints(predictors, 10000)
# Create SWD object
presence <- prepareSWD(species = "Vultur gryphus", coords = p_coords, env = predictors, categorical = "biome")
bg <- prepareSWD(species = "Vultur gryphus", coords = bg_coords, env = predictors, categorical = "biome")
# Train a model
sdmtune_model <- train(method = "Maxent", p = presence, a = bg)
```

We want to compare the execution time of the `predict` function between
**SDMtune** that uses its own algorithm and **dismo** (Hijmans et al.
2017) that calls the MaxEnt Java software. We first convert the
`sdmtune_model` in a object that is accepted by **dismo**:

``` r
maxent_model <- SDMmodel2MaxEnt(sdmtune_model)
```

Here a function to test that the results are equal, with a tolerance of
`1e-7`:

``` r
my_check <- function(values) {
  return(all.equal(values[[1]], values[[2]], tolerance = 1e-7))
}
```

Now we test the execution time using the **microbenckmark** package:

``` r
bench <- microbenchmark::microbenchmark(
  "SDMtune" = predict(sdmtune_model, data = predictors, type = "cloglog"),
  "dismo" = predict(maxent_model, predictors),
  check = my_check
)
```

Plot the output:

``` r
library(ggplot2)
ggplot(bench, aes(x = expr, y = time/1000000, fill = expr)) +
  geom_boxplot() +
  labs(fill = "", x = "Package", y = "time (milliseconds)") +
  theme_minimal()
```

<div style="text-align: center">

<img src="man/figures/bench.png" alt="" />

</div>

The execution time is in average about two time faster\!

## Code of conduct

Please note that this project follows a [Contributor Code of
Conduct](.github/CODE_OF_CONDUCT.md).

### References

<div id="refs" class="references">

<div id="ref-Hijmans2017">

Hijmans, Robert J., Steven Phillips, John Leathwick, and Jane Elith.
2017. “dismo: Species Distribution Modeling.”
<https://cran.r-project.org/package=dismo>.

</div>

</div>
