---
title: Oracle executeBatch异常：ArrayIndexOutOfBoundsException
date: 2018-11-18 17:16:00 +0800
updated: 2018-11-18 17:16:00 +0800
comments: false
---
## 问题
在使用 JDBC 的 `PreparedStatement.executeBatch()` 接口向 Oracle 数据库批量插入数据时，出现数组越界异常。
## 解决过程
通过观察异常信息，是 Oralce JDBC 驱动的内部方法出现了数组越界异常。为了搞清楚实际原因，在社区找到了一篇[帖子](https://community.oracle.com/thread/599441)。其中的一个回复写道：
> The 10g driver apparently keeps a global serial number for all parameters in the entire batch, with a "short" variable. So you can have at most 32768 parameters in the batch. I was having the same exception because I have a INSERT statement with 42 parameters and my batches can be as big as 1000 records, so 42000 > 32768 and this overflows to a negative index. I reduced the batch factor to 100 to be safe, and all is well. I guess your update DML should have a larger number of parameters per record, right? (My diagnostic of the bug is just deduction from the symptoms)

大体的意思是，Oracle 的 `preparedStatement` 批量执行 SQL 时，对参数个数是有上限的（针对不同版本的oracle驱动，这个上限对不同的可能是不同的），这个参数个数的含义指 `addBatch` 的次数×每条 SQL 中的参数个数。对于 Oracle 10g 的驱动来说，这个值可能是32,768，所以编程时参数个数应该小于这个值，否则报错。

## 方案
在我的项目中，每 10,000 条提交一次，每一条中有 27 个参数。也就是说总共有 270,000 个参数，这远远超出了上限。降低 `addBatch` 的数量（如每 1,000 条提交一次）即可解决异常。
