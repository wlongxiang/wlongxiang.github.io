---
layout: post
comments: true
title:  "Pyspark: groupby, aggregate and window operations"
excerpt: "A simple tutorial on groupBy, aggregate and window operations in pyspark."
date:   2019-12-30 9:00:00
mathjax: true
---

In this blog, in the first part, we are gonna walk through the `groupBy` and aggregation operation in spark with ready to
run code samples. Then in the second part, we aim to shed some lights on the the powerful *window* operation.

## What is `groupby`?
The `groupBy` function allows you to group rows into a so-called *Frame* which has same values of certain column(s).
`groupBy` operation is almost always used together with aggregation functions.

In spark, the [`DataFrame.groupBy(*cols)`](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.DataFrame.groupBy) API, returns a `GroupedData` object, on which aggregation functions can
be applied. Below is a list of builtin aggregations:

- *avg, max, min, sum, count*

Note that it is possible to define your own aggregation functions using [pandas_udf](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.functions.pandas_udf).
We will cover it at another time.

### Code example (ready to run)
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
To show total salary for each department:
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
To calculate the budget for the whole company, we can omit column name in `groupBy` operation:
```python
# sum of salaries for all departments

df.groupBy().sum("salary").show()
>>>
+-----------+
|sum(salary)|
+-----------+
|       9300|
+-----------+

# for simply aggregations without grouping, one can use sql functions as shorthand
from pyspark.sql.function import *
df.select(sum("salary")).show()
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

## What is `window`?

Window functions operate on a set of rows and return a single value for each row. This is different
than the `groupBy` and aggregation function in part 1, which only returns a single value for each group
or Frame.


The window function is spark is largely the same as in traditional SQL with `OVER()` clause.
The `OVER()` clause has the following capabilities:
                                                                                             
- Defines window partitions to form groups of rows. (PARTITION BY clause)
- Orders rows within a partition. (ORDER BY clause)

NB: `PARTIION BY` is similar to `groupBy` as discussed before in a sense that both create groups or rows or
Frames.

A normal aggregation function returns a single value:
```python
# register the table
df.createOrReplaceTempView("salary")
spark.sql("select avg(salary) as avg_salary_company from salary").show()
#OR equivalently with df.groupBy().avg("salary").show()
>>>
+------------------+
|avg_salary_company|
+------------------+
|            1860.0|
+------------------+
```
A window function with over returns a value for each row:
```python
spark.sql("select name, department, avg(salary) over() as avg_salary_company from salary").show()
>>>
2019-12-30 14:58:06,763 WARN window.WindowExec: No Partition Defined for Window operation! Moving all data to a single partition, this can cause serious performance degradation.
+------+----------+------------------+
|  name|department|avg_salary_company|
+------+----------+------------------+
| alice|        HR|            1860.0|
|   bob|        IT|            1860.0|
| clair|        IT|            1860.0|
|stefan|   Finance|            1860.0|
|darren|        HR|            1860.0|
+------+----------+------------------+
```
NB: the warning says that we we should use partion inside the `OVER()` clause to avoid collecting
all rows into a single partition.

## How to define window operations in spark?

There are three parts for a window specification (see reference [2]):

- Partitioning Specification (`PARTITION BY`): controls which rows will be in the same partition with the given row. Also, the user might want to make sure all rows having the same value for  the category column are collected to the same machine before ordering and calculating the frame.  If no partitioning specification is given, then all data must be collected to a single machine.
- Ordering Specification (`ORDER BY`): controls the way that rows in a partition are ordered, determining the position of the given row in its partition.
- Frame Specification (`{ RANGE | ROWS } BETWEEN frame_start AND frame_end`): states which rows will be included in the frame for the current input row, based on their relative position to the current row.  For example, “the three rows preceding the current row to the current row” describes a frame including the current input row and three rows appearing before the current row.

In pyspark, we can specify window definition as shown below, equivalent to `Over(PARTITION BY COL_A ORDER BY COL_B ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)` in SQL.

In this example, we create a fully qualified window specification with all three parts, and
calculate the average salary per department:
```python
from pyspark.sql.window import Window
window_spec = Window.partitionBy("department").orderBy("name").rowsBetween(Window.unboundedPreceding, Window.unboundedFollowing)
df.select(df.department, avg("salary").over(windowSpec).alias("avg_salary_depart")).show()
>>>
+----------+-----------------+
|department|avg_salary_depart|
+----------+-----------------+
|        HR|           1050.0|
|        HR|           1050.0|
|   Finance|           3000.0|
|        IT|           2100.0|
|        IT|           2100.0|
+----------+-----------------+
```

The above code is the same as:
```python
spark.sql("SELECT department, avg(salary) OVER(PARTITION BY department ORBER BY name ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) as avg_salary_depart from salary").show()
>>>
+----------+-----------------+
|department|avg_salary_depart|
+----------+-----------------+
|        HR|           1050.0|
|        HR|           1050.0|
|   Finance|           3000.0|
|        IT|           2100.0|
|        IT|           2100.0|
+----------+-----------------+
```

## References
[1. A gentle introduction into window function in traditional SQL](https://drill.apache.org/docs/sql-window-functions-introduction/)

[2. This blog from databricks explains the window feature in spark very well](https://databricks.com/blog/2015/07/15/introducing-window-functions-in-spark-sql.html).

















