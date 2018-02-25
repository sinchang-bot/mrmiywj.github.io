---
layout: post
title: Spark optimization
category: data
comments: true
tags: data ml cmu
---
I'm taking a course called Advanced Cloud Computing at CMU this semester. The course project, Machine Learning on Spark taught me a lot on how to write correct and efficient Spark code. Here are some experiences.

# Python code optimization

Since the instructors asked us to write program in Python, the python code's optimization should be considered first. For example, you should convert your valid word lists to sets, which will dramatically improve the time complexity from *O(n)* to *O(1)*. Another example is to use string's *join* or *format* over string concatenation.


# Use broadcast variables widely

We should use broadcast variables if the variables are read only. The reason is that broadcast variables are spread to all the nodes by P2P file transfer. If not, it will be wrapped into the closure and be sent by RPC to all nodes, which is slower.

But the broadcast is not always suitable in every situation. If the variables are too large to fit into the memory, we should try other approaches.

# GroupByKey vs reduceByKey

There are some articles declaiming that `You should always use reduceByKey other than groupByKey`, like [here](https://github.com/vaquarkhan/vk-wiki-notes/wiki/reduceByKey--vs-groupBykey-vs-aggregateByKey-vs-combineByKey). This conclusion is right under almost every situation. Since there are few operations which need to group the values. The common situation is some computations over the values. But if you actually need to group all the values, for example build an inverted index, you should use `groupByKey`.

# Avoid using tuple as values

I'm not sure if it is a universal rule. But I encounter this issue when I want to group the result of several computations over the values of the same key when I am calling `reduceByKey`. But this caused a weak performance. I think this is due to the cost of serialization of tuple is much more than primitive types.

# Join

Join is commonly considered as a expansive operation. But it is really useful when you join two RDDs based on same partitioning. This is really important when I did `join-based parameter communication` in the gradient descending.

To achieve this, we need to understand how Spark partition its RDDs over nodes. There are some basic rules:

1. The <Key, Value> pair is partitioned based on the key's value
2. Some functions which has a parameter [partition function](http://spark.apache.org/docs/2.2.0/api/python/pyspark.html) will repartition/ coalesce the RDDs. If you do not pass any specific functions, the default functions are the same.

Thus we can infer that if we partition the RDDs which has same type of keys into same number of parts, they will be aligned. It is important for join operations.
