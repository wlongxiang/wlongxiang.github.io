---
layout: post
comments: true
title:  "Pyspark: groupBy, aggregate and window operations"
excerpt: "A simple tutorial on groupBy, aggregate and window operations in pyspark."
date:   2019-12-30 9:00:00
mathjax: true
---

## What is `groupBy`?
The `groupBy` function allows you to group rows (observations in ML terms) which has same values of certain column(s).
`groupBy` operation is almost always used together with aggregation functions.

In spark, the `DataFrame.groupBy(*cols)` API, returns a `GroupedData` object, on which aggregation functions can
be applied. Below is a list of builtin aggregations:

- *avg, max, min, sum, count*

Note that it is possible to define your own aggregation functions using [pandas_udf](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.functions.pandas_udf).
We will cover it at another time.

### Code example
We first create some dummy salary data.
```python
import pandas as pd
pddf = pd.DataFrame({"department": ["HR", "IT", "IT", "Finance", "HR"], 
                    "Name": ["alice", "bob", "clair", "stefan", "darren"], 
                    "salary": [1000, 2000, 2200, 3000, 1100],
                    "job_level": [1, 2, 2, 3, 1]})
df = spark.createDataFrame(pddf)
df.show()
>>>
+------+----------+---------+------+
|  Name|department|job_level|salary|
+------+----------+---------+------+
| alice|        HR|        1|  1000|
|   bob|        IT|        2|  2000|
| clair|        IT|        2|  2200|
|stefan|   Finance|        3|  3000|
|darren|        HR|        1|  1100|
+------+----------+---------+------+
```
We can use `groupBy` and builtin aggregation functions to calculate number of employees at different
departments, average salary for different departments, etc. Let's see how we can achieve this with a few lines of code.


```python
# average salary for different departments

df.groupBy("department").avg("salary").show()
>>>
+----------+-----------+
|department|avg(salary)|
+----------+-----------+
|        HR|     1050.0|
|   Finance|     3000.0|
|        IT|     2100.0|
+----------+-----------+
```
To show the headcount budget:
```python
# sum of salaries for different departments

df.groupBy("department").sum("salary").show()
>>>
+----------+-----------+
|department|sum(salary)|
+----------+-----------+
|        HR|       2100|
|   Finance|       3000|
|        IT|       4200|
+----------+-----------+
```
To calculate the budget for whole company, we can omit column name in `groupBy` operation:
```python
# sum of salaries for all departments

df.groupBy().sum("salary").show()
>>>
+-----------+
|sum(salary)|
+-----------+
|       9300|
+-----------+
```

To calculate number of employees in each department:
```python
# count of people for different departments
df.groupBy("department").count().show()
>>>
+----------+-----+
|department|count|
+----------+-----+
|        HR|    2|
|   Finance|    1|
|        IT|    2|
+----------+-----+
```
**NB: count() does not take arguments, because it simply returns number of rows in each group.**









