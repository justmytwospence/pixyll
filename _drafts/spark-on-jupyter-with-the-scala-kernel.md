---
author: Spencer Boucher
layout: post
summary: Use the jupyter-scala kernel to play with Spark
title: Spark on Jupyter with the Scala kernel
---

When you are writing involved data transformations in your Spark app, having a
REPL can make things easier by an order of magnitude. The spark-shell is pretty
good for this, but sometimes you also really want to *visualize* the
transformations you are making. This is where the Jupyter notebook comes in
handy. There are some fancy Jupyter kernels out there that hook into a Spark
cluster for you, but its not that hard to just set up a plain old Scala Jupyter
kernel either.

```scala
load.resolver("DefaultMavenRepository" at "https://repo1.maven.org/maven2")
load.ivy("org.apache.spark" % "spark-core_2.11" % "1.6.0")
load.jar("/home/sboucher/spark-bin/michelangelo*")
```
