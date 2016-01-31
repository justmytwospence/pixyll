---
author: Spencer Boucher
categories:
date: 2014-01-10
layout: post
summary:
title: Stratified sampling in R
---

I was surprised to find that R doesn’t have a base function for stratified
random sampling. There’s not even a well known package I could find that does
this in a straight forward way. So heres my own.

It is essentially a wrapper for a ddply call that samples each subset and then
combines them. If the size argument is less than 1, it will be interpreted as
the percentage of each stratification subset that should be sampled. If the size
argument is greater than 1, it will be interpreted as the number of observations
to sample from each stratification subset.

Note that in the first case, a different number of observations will be taken
from each subset depending on their total number of observations. In the second
case however, an equal number of observations will be sampled from each subset,
regardless of their total number of observations.

The .by argument is formulated the same way it is for any other ddply call.

```R
stratified_sample <- function(df, size = .5, .by, seed = 37L) {
  require(plyr)
  set.seed(seed)
  df.sample <- ddply(df, .by,
                     function(x) {
                       if (size < 1) { size <- size * nrow(x) }
                       return(x[sample(nrow(x), size = size),])
                     },
                     .progress = 'text')
  return(df.sample)
}
```
