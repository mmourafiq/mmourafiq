---
layout: blog
title: "Spark context manager"
subtitle: "a context manager to communicate with pyspark."
cover_image: images/blog/spark.png
cover_image_caption: ""
tags: [spark, python]
---
park is data processing and clustering framework that allows developers to create parallel apps in Scala, Java, and Python.

For Python developers, Spark exposes it's programming model through a Python API ([pyspark](https://spark.apache.org/docs/latest/api/python/index.html)). And in order to communicate with Spark from a python application, let's say django, we need context during this communication.

First step check that `SPARK_HOME` is set, otherwise you need to do it.

If you are using a virtualenv for you python project, you need to add spark to this virtualenv:

```bash
>  add2virtualenv path-to-spark/python
>  add2virtualenv path-to-spark/python/lib/py4j-0.8.2.1-src.zip
```

 This will add the spark to your virtualenv python path. You can check that `_virtualenv_path_extensions.pth` contains this new entries.

```bash
>  path-to-virtualenv/lib/python2.7/site-packages/_virtualenv_path_extensions.pth
```

Now we are ready to add a spark context manager:

```python
import contextlib

from pyspark import SparkContext, SparkConf

SPARK_MASTER = 'local'
SPARK_APP_NAME = 'app'

@contextlib.contextmanager
def spark_manager(spark_master=SPARK_MASTER,
                  spark_app_name=SPARK_APP_NAME,
                  spark_executor_memory=None):

    conf = SparkConf().setMaster(spark_master)
    conf.setAppName(spark_app_name)

    if spark_executor_memory:
        conf.set("spark.executor.memory", spark_executor_memory)

    spark_context = SparkContext(conf=conf)

    try:
        yield spark_context
    finally:
        spark_context.stop()
```

And you can use this context manager for spark related jobs, you only need to wrap in a `with` statement.

```python
with spark_manager() as context:
    l = context.parallelize([1, 3, 4, 5]).collect()
```
