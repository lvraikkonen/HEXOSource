---
title: SQL Server表分区
date: 2017-06-06 17:53:27
tags:
  - SQL Server
  - Database

---


分区表是把数据按某种标准划分成区域存储在不同的文件组中，使用分区可以快速而有效地管理和访问数据子集，从而使大型表或索引更易于管理。合理的使用分区会很大程度上提高数据库的性能。

使用表分区最主要场景是用于:

- 存档，比如将销售记录中1年前的数据分到一个专门存档的服务器中
- 便于管理，比如把一个大表分成若干个小表，则备份和恢复的时候不再需要备份整个表，可以单独备份分区
- 提高可用性，当一个分区跪了以后，只有一个分区不可用，其它分区不受影响
- 提高性能，这个往往是大多数人分区的目的，把一个表分布到不同的硬盘或其他存储介质中，会大大提升查询的速度.

创建分区需要如下个步骤：
1. 创建文件组
2. 创建分区函数
3. 创建分区方案
4. 创建或者修改使用分区方案的表


<!-- more -->

举一个按照时间分区的案例：

确定分区键列的类型(DATETIME)以及分区的边界值:
- 2010-01-01
- 2012-01-01
- 2014-01-01

N个边界值确定 N+1 个分区

![BorderValue](http://7xkfga.com1.z0.glb.clouddn.com/borderValue.png)

## 创建文件组

T-SQL语法

```
alter database <数据库名> add filegroup <文件组名>
```

下面创建4个分区文件组

``` sql
USE AdventureWorks2012;  
GO  

-- Adds four new filegroups to the AdventureWorks2012 database  
ALTER DATABASE AdventureWorks2012  
ADD FILEGROUP test1fg;  
GO  
ALTER DATABASE AdventureWorks2012  
ADD FILEGROUP test2fg;  
GO  
ALTER DATABASE AdventureWorks2012  
ADD FILEGROUP test3fg;  
GO  
ALTER DATABASE AdventureWorks2012  
ADD FILEGROUP test4fg;   

-- Adds one file for each filegroup.  
ALTER DATABASE AdventureWorks2012   
ADD FILE   
(  
    NAME = test1dat1,  
    FILENAME = 'E:\DB\t1dat1.ndf',  
    SIZE = 5MB,  
    MAXSIZE = 100MB,  
    FILEGROWTH = 5MB  
)  
TO FILEGROUP test1fg;  
ALTER DATABASE AdventureWorks2012   
ADD FILE   
(  
    NAME = test2dat2,  
    FILENAME = 'E:\DB\t2dat2.ndf',  
    SIZE = 5MB,  
    MAXSIZE = 100MB,  
    FILEGROWTH = 5MB  
)  
TO FILEGROUP test2fg;  
GO  
ALTER DATABASE AdventureWorks2012   
ADD FILE   
(  
    NAME = test3dat3,  
    FILENAME = 'E:\DB\t3dat3.ndf',  
    SIZE = 5MB,  
    MAXSIZE = 100MB,  
    FILEGROWTH = 5MB  
)  
TO FILEGROUP test3fg;  
GO  
ALTER DATABASE AdventureWorks2012   
ADD FILE   
(  
    NAME = test4dat4,  
    FILENAME = 'E:\DB\t4dat4.ndf',  
    SIZE = 5MB,  
    MAXSIZE = 100MB,  
    FILEGROWTH = 5MB  
)  
TO FILEGROUP test4fg;  
GO  
```

执行上面的脚本，可以创建4个文件组

## 创建分区函数

指定分依据区列（依据列唯一），分区数据范围规则，分区数量，然后将数据映射到一组分区上。

创建语法： 

```
create partition function 分区函数名(<分区列类型>) as range [left/right] 
for values (每个分区的边界值,....) 
```

``` sql
-- Creates a partition function called myRangePF1 that will partition a table into four partitions  
CREATE PARTITION FUNCTION myRangePF1 (datetime)  
    AS RANGE LEFT FOR VALUES ('2010-01-01','2012-01-01','2014-01-01') ;  
GO  
```

左边界/右边界：就是把临界值划分给上一个分区还是下一个分区。一个小于号，一个小于等于号。

> 注意：只有没有应用到分区方案中的分区函数才能被删除。

## 创建分区方案

指定分区对应的文件组。

创建语法： 

```
-- 创建分区方案语法
create partition scheme <分区方案名称> as partition <分区函数名称> [all]to (文件组名称,....) 
```

``` sql
-- Creates a partition scheme called myRangePS1 that applies myRangePF1 to the four filegroups created above  
CREATE PARTITION SCHEME myRangePS1  
    AS PARTITION myRangePF1  
    TO (test1fg, test2fg, test3fg, test4fg) ;  
GO  
```

> 注意：只有没有分区表，或索引使用该分区方案是，才能对其删除。

## 创建使用分区方案的表

创建语法：

```
--创建分区表语法
create table <表名> (
  <列定义>
)on<分区方案名>(分区列名)
```

``` sql
-- Creates a partitioned table called PartitionTable that uses myRangePS1 to partition col1  
CREATE TABLE [dbo].[Data_partitioned](  
    [Id] bigint  NOT NULL,  
    [Name] [varchar](10)  NULL,  
    [CreateDate] [datetime] NOT NULL
)  ON [myRangePS1] (CreateDate);   
-- 这里使用[myRangePS1]架构，根据CreateDate列进行分区 
```

插入一些测试数据，看看是不是按照给定的分区方案将数据分区了

``` sql
-- 查看使用情况   
SELECT *, $PARTITION.myRangePF1([CreateDate])  as groupName
FROM dbo.Data_partitioned 
```

**在创建分区表后，需要创建聚集分区索引**

根据订单表Orders 查询时经常使用OrderDate 范围条件来查询的特点，我们最好在Orders.OrderDate 列上建立聚集索引（clustered index）。为了便于进行分区切换（partition swtich)。
大多数情况下，建议在分区表上建立分区索引。


## 查询分区表信息

### 查看分区依据列的指定值所在的分区

``` sql
-- 查询分区依据列为'2013-01-01'的数据在哪个分区上
SELECT $partition.myRangePF1('2013-01-01')  
-- 返回值是3，表示此值存在第2个分区 
```

### 查询每个非空分区存在的行数

``` sql
-- 查看分区表中，每个非空分区存在的行数
SELECT $partition.myRangePF1(CreateDate) as partitionNum
     , count(*) as recordCount
FROM dbo.Data_partitioned
GROUP BY $partition.myRangePF1(CreateDate)
```

![partitionRowCount](http://7xkfga.com1.z0.glb.clouddn.com/rowCount.png)

## 分区的拆分合并以及数据移动

### 拆分分区

在分区函数中新增一个边界值，即可将一个分区变为2个。

``` sql
--分区拆分
alter partition function myRangePF1()
split range('2011-01-01')  --将第二个分区拆为2个分区
```

> 注意：如果分区函数已经指定了分区方案，则分区数需要和分区方案中指定的文件组个数保持对应一致。


### 合并分区

与拆分分区相反，去除一个边界值即可。

``` sql
--合并分区
alter partition function myRangePF1()
merge range('2011-01-01')  --将第二第三分区合并
```

### 分区数据移动

可以使用 `ALTER TABLE ....... SWITCH` 语句快速有效地移动数据子集：

- 将某个表中的数据移动到另一个表中；
- 将某个表作为分区添加到现存的已分区表中；
- 将分区从一个已分区表切换到另一个已分区表；
- 删除分区以形成单个表。