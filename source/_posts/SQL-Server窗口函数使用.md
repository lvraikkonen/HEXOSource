---
title: SQL Server窗口函数使用
tags:
  - SQL
  - SQL Server
date: 2017-06-12 15:43:28
---


## 什么是窗口函数 Windows Function

窗口函数属于集合函数，作用在行集上，下面这段关于窗口函数的介绍来自 [PostgreSQL intro windows function](https://www.postgresql.org/docs/9.1/static/tutorial-window.html)

> A window function performs a calculation across a set of table rows that are somehow related to the current row. This is comparable to the type of calculation that can be done with an aggregate function. But unlike regular aggregate functions, use of a window function does not cause rows to become grouped into a single output row — the rows retain their separate identities. Behind the scenes, the window function is able to access more than just the current row of the query result.


窗口函数在SQL:2003标准中被添加，并在SQL:2008标准中被细化。传统的关系型数据库：Oracle、Sybase和DB2都已经支持窗口函数，像开源的PostgreSQL里面也已经有了对窗口函数的完整的实现。SQL Server 2005开始对窗口函数有了最初的支持，从SQL Server 2012开始，窗口函数也被SQL Server完全支持。

<!-- more -->

## SQL Server窗口函数

窗口函数的应用非常广泛，像分页、去重、分组的基础上返回 Top N 的行、计算 Running Totals、Gaps and islands、百分率, Hierarchy 排序、Pivoting 等等。

窗口函数是整个SQL语句最后被执行的部分，这意味着窗口函数是在SQL查询的结果集上进行的，因此不会受到Group By， Having，Where子句的影响

SQL Server 窗口函数主要用来处理由 OVER 子句定义的行集, 主要用来分析和处理

- Running totals
- Moving averages
- Gaps and islands


在标准的SQL中，Window Function 的OVER语句中有三个非常重要的元素:
- Partitioning
- Ordering
- Framing

这三种元素的作用可以限制窗口集中的行，如果没有指定任何元素，那么窗口中包含的就是查询结果集中所有的行。

窗口函数的语法：

```
-- Syntax for SQL Server, Azure SQL Database, and Azure SQL Data Warehouse  

OVER (   
       [ <PARTITION BY clause> ]  
       [ <ORDER BY clause> ]   
       [ <ROW or RANGE clause> ]  
      ) 
```

### Partition

> Divides the query result set into partitions. The window function is applied to each partition separately and computation restarts for each partition.

通过PARTITION BY 得到的窗口集是基于当前查询结果的当前行的一个集,，比如说 PARTITION BY CustomerID，当前行的 CustomerID = 1，那么对于当前行的这个 Window 集就是在当前查询结果之上再加上 CustomerID = 1 的一个查询结果。


### Order

> Defines the logical order of the rows within each partition of the result set. That is, it specifies the logical order in which the window function calculation is performed.

Order By子句对于诸如Row_Number()，Rank()，Lead()，LAG()等函数是必须的，因为如果数据无序，这些函数的结果就没有任何意义

### ROW / RANGE

> Further limits the rows within the partition by specifying start and end points within the partition. This is done by specifying a range of rows with respect to the current row either by logical association or physical association. Physical association is achieved by using the ROWS clause.

## 简单的例子

下面用一个简单的例子表示传统的聚合函数和窗口函数的区别

有一个需求：将AdventureWorks示例数据库中的Employee表按照性别进行聚合，希望得到的结果是："登录名，性别，该性别所有员工的总数"

那么传统的写法是用子查询获得按照性别进行聚合的值，然后再关联

``` sql
SELECT  [LoginID]
      , [Gender]
      , (SELECT COUNT(*) FROM [AdventureWorks2012].[HumanResources].[Employee] a WHERE a.Gender=b.Gender) AS GenderTotal
FROM [AdventureWorks2012].[HumanResources].[Employee] b
```

如果使用窗口函数完成这个功能，代码如下：

``` sql
SELECT [LoginID]
     , [Gender]
     , COUNT(*) OVER(PARTITION BY Gender) AS GenderTotal
FROM [AdventureWorks2012].[HumanResources].[Employee]
```

![ExecutionPlanCompare](http://7xkfga.com1.z0.glb.clouddn.com/executionCompare1.png)


## 窗口函数与 Group, 子查询语句的比较

对于Group来说，SELECT语句中的列必须是Group子句中出现的列或者是聚合列，那么如果需要同时在 SELECT 语句中查询其它的非 Group 或者非聚合列, 那么就需要额外的子查询。

一个和上面例子很相似的情景，比如要查询每个客户的每个订单的值，以及这个订单于这个订单客户的所有订单总和比，以及这个订单与这个客户所有订单平均值的差。

一个SELECT语句肯定是搞不定的，如下面代码：

``` sql
WITH Aggregates AS
(
   SELECT custid
        , SUM(val) AS sumval
        , AVG(val) AS avgval
   FROM Sales.OrderValues
   GROUP BY custid
)
SELECT O.orderid
     , O.custid
     , O.val
     , CAST(100. * O.val / A.sumval AS NUMERIC(5, 2)) AS pctcust
     , O.val - A.avgval AS diffcust
FROM Sales.OrderValues AS O
JOIN Aggregates AS A
ON O.custid = A.custid;
```

因为没有办法在一个Group查询中同时显示 Detail和汇总的信息

如果这时再加一个比 - 单个订单与总订单额/平均额比，这时汇总的级别又不相同了， 需要单独再汇总一次

额~ 又要添加一层子查询聚合

如果提出更多的聚合和比较，查询语句会越来越复杂，并且查询优化器也不能确定每次是否都访问的是同一个数据集，因此需要分别访问数据集，造成性能下降。

通过使用窗口函数可以很容易解决这些问题，因为可以为每一种聚合定义一个窗口上下文。

``` sql
SELECT orderid
     , custid
     , val
     , CAST(100.* val/ SUM(val) OVER(PARTITION BY custid) AS NUMERIC(5,2)) AS pctcut
     , val - AVG(val) OVER(PARTITION BY custid) AS diffcust
     , CAST(100.* val/ SUM(val) OVER() AS NUMERIC(5,2)) AS pctall
     , val - AVG(val) OVER() AS diffall
FROM Sales.OrderValues
```