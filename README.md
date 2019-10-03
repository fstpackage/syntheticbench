
<!-- README.md is generated from README.Rmd. Please edit that file -->

<!-- <img src="logo.png" align="right" /> -->

[![Linux/OSX Build
Status](https://travis-ci.org/fstpackage/syntheticbench.svg?branch=develop)](https://travis-ci.org/fstpackage/syntheticbench)
[![Windows Build
status](https://ci.appveyor.com/api/projects/status/rng88laj6o2fj2dy?svg=true)](https://ci.appveyor.com/project/fstpackage/syntheticbench)
[![License: AGPL
v3](https://img.shields.io/badge/License-AGPL%20v3-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)
[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-blue.svg)](https://www.tidyverse.org/lifecycle/#experimental)
[![codecov](https://codecov.io/gh/fstpackage/syntheticbench/branch/develop/graph/badge.svg)](https://codecov.io/gh/fstpackage/syntheticbench)

## Overview

The `syntheticbenchmark` package provides tooling to greatly symplify
benchmarking of serialization solutions such as `fst`, `arrow`,
`feather` or `sqlite`. By using a standardized method of benchmarking,
results become more reliable and more easy to compare.

## Features

Some of the optimizations used in `syntheticbench`:

## Reproducing benchmark on fst homepage using syntheticbench

``` r
library(syntheticbench)
library(fst)
library(arrow)

# medium sized dataset
nr_of_rows <- 1e7

# generator for 'fst benchmark' dataset
generator <- table_generator(
  "fst benchmark",
  function(nr_of_rows) {
    data.frame(
      Logical = sample_logical(nr_of_rows, true_false_na_ratio = c(85, 10, 5)),
      Integer = sample_integer(nr_of_rows, max_value = 100L),
      Real    = sample_integer(nr_of_rows, 1, 10000, max_distict_values = 20) / 100,
      Factor  = as.factor(sample(labels(UScitiesD), nr_of_rows, replace = TRUE))
    )}
)


# baseR streamer
rds_streamer <- table_streamer(
  id = "rds",
  table_writer = function(x, file_name, compress) {
    saveRDS(x, file_name)
  },
  table_reader = function(x) readRDS(x),
  can_select_threads = FALSE,
  variable_compression = FALSE
)


# fst streamer
fst_streamer <- table_streamer(
  id = "fst",
  table_writer = function(x, file_name, compress) {
    if (is.null(compress)) {
      return(fst::write_fst(x, file_name))
    }
    fst::write_fst(x, file_name, compress)
  },
  table_reader = function(x) read_fst(x),
  can_select_threads = TRUE,
  variable_compression = TRUE
)


# parguet streamer
parguet_streamer <- table_streamer(
  id = "parguet",
  table_writer = function(x, file_name, compress) {
    arrow::write_parquet(x, file_name)
  },
  table_reader = function(x) read_parquet(x),
  can_select_threads = FALSE,
  variable_compression = FALSE
)


# feather streamer
feather_streamer <- table_streamer(
  id = "feather",
  table_writer = function(x, file_name, compress) {
    arrow::write_feather(x, file_name)
  },
  table_reader = function(x) read_feather(x),
  can_select_threads = FALSE,
  variable_compression = FALSE
)


table_streamers <- list(rds_streamer, fst_streamer, parguet_streamer, feather_streamer)


bench_result <- syntheticbench::synthetic_bench(
  generator,
  table_streamers,
  1e7
)
```
