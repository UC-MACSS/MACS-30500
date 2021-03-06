---
title: "Build a better workflow"
date: 2021-07-15T09:30:00-06:00
publishDate: 2019-05-06T09:30:00-06:00
draft: false

# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
time_end: 2021-07-15T11:30:00-06:00
all_day: false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors: []

# Abstract and optional shortened version.
abstract: ""
summary: "Introduce and practice techniques for preprocessing data sets and developing model workflows using the `tidymodels` framework."

# Location of event.
location: "Online"

# Is this a selected talk? (true/false)
selected: false

# Tags (optional).
#   Set `tags: []` for no tags, or use the form `tags: ["A Tag", "Another Tag"]` for one or more tags.
tags: []

# Links (optional).
url_pdf: ""
url_slides: "/slides/build-a-better-workflow/"
url_video: ""
url_code: ""

# Does the content use math formatting?
math: false
---



## Overview

* Identify the importance of preprocessing data sets
* Introduce the `recipes` package for preprocessing data
* Utilize `usemodels` to automatically construct code templates for common model types
* Construct workflows for machine learning

## Before class

* Read [Preprocess your data](/notes/preprocess/)
* Read [Tune model parameters](/notes/tune-models/)

This is not a math/stats class. In class we will **briefly** summarize how these methods work and spend the bulk of our time on estimating and interpreting these models. That said, you should have some understanding of the mathematical underpinnings of statistical learning methods prior to implementing them yourselves. See below for some recommended readings:

* Chapter 5 in [*An Introduction to Statistical Learning*](http://link.springer.com.proxy.uchicago.edu/book/10.1007%2F978-1-4614-7138-7)
* Chapters 2-3 in [*Hands-On Machine Learning with R*](https://bradleyboehmke.github.io/HOML/)
* [*Feature Engineering and Selection: A Practical Approach for Predictive Models*](http://www.feat.engineering/)

## Class materials

{{% callout note %}}

Run the code below in your console to download the exercises for today.

```r
usethis::use_course("uc-cfss/machine-learning")
```

{{% /callout %}}

{{% callout note %}}

Materials derived from [Tidymodels, Virtually: An Introduction to Machine Learning with Tidymodels](https://tmv.netlify.app/site/) by [Allison Hill](https://alison.rbind.io/).

{{% /callout %}}

### Additional readings

* [`caret`](https://topepo.github.io/caret/) 
* [`tidymodels`](https://www.tidymodels.org/start/)
* [*Tidy Modeling with R*](https://www.tmwr.org/) - a book-length introduction to tidy modeling in R. Still an early work-in-progress

## What you need to do after class

* [Complete the machine learning homework](/homework/machine-learning/)
