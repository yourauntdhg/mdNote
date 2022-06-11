# 第一章 Hive基本概念

## 1.1 什么是Hive

1. Hive简介

   Hive：由Facebook开源用于解决海量结构化日志的数据统计工具。Hive是基于Hadoop的一个<font color=red>数据仓库工具</font>，可以将<font color=red>结构化的数据文件映射为一张表</font>，并提供<font color=red>类SQL</font>查询功能。

2. Hive本质：将HQL转化为MapReduce程序。

   - Hive处理的数据存储在HDFS。
   - Hive分析数据底层的实现是MapReduce。
   - 执行程序运行在yarn上。

```mermaid
graph LR
	A["数据仓库通过SQL<br>进行统计分析"]-->B("将SQL语言中常用的操作<br>(select,where,group等)<br>用MapReduce写成很多模板")
	B-->C(("所有的<br>MapReduce<br>模板封装<br>在Hive中"))
	C-->D(["通过Hive框架<br>匹配出相应的<br>MapReduce模板"])
	D-->E(["运行MapReduce<br>程序，生成相应<br>的分析结果"])
	E-->F(("result"))
	F-->G(("client"))
	G-->H(["用户根据<br>业务需求<br>编写相应<br>的SQL语句"])
	H-->C
```

## 1.2 Hive的优缺点

1. 优点
   1. 操作接口采用SQL语法，提供快速开发的能力（简单，易上手）。
   2. 避免写MapReduce，减少开发人员的学习成本。
   3. Hive的执行延迟比较高，因此Hive常用于数据分析，对实时性要求不高的场合。
   4. Hive优势在于处理大数据，对于处理小数据没有优势，因为Hive的执行延迟比较高。
   5. Hive支持用户自定义函数，用户可以根据自己的需求来实现自己的函数。
2. 缺点
   1. Hive的HQL表达能力有限
      - 迭代式算法无法表达。
      - 数据挖掘方面不擅长，由于MapReduce数据处理流程的限制，效率更高的算法啊却无法实现。
   2. Hive的效率比较低
      - Hive自动生成的MapReduce作业，通常不够智能化。
      - Hive调优比较困难，粒度较粗。

## 1.3 Hive架构原理

![Hive架构](../img/Hive架构图.jpeg)

1. 用户接口：Client

   CLI(command-line interface)、JDBC\ODBC（jdbc访问hive）、WEBUI（浏览器访问hive）。

2. 元数据：Metastore

   元数据包括：表名、表所属的数据库（默认是default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录；

   <font color=red>默认存储在自带的derby数据库中，推荐使用MySQL存储Metastore。</font>

3. Hadoop

   使用HDFS进行存储，使用MapReduce进行计算。

4. 驱动器：Driver

   1. 解析器（SQL Parser）：将SQL字符串转换成抽象语法树AST，这一步一般都用第三方工具库完成，比如antlr；对AST进行语法分析，比如表是否存在、字段是否存在、SQL语义是否有误。
   2. 编译器（Physical Plan）：将AST编译生成逻辑执行计划。
   3. 优化器（Query Optimizer）：对逻辑执行计划进行优化。
   4. 执行器（Execution）：把逻辑执行计划转换成可以运行的物理计划。对于Hive来说就是MR/Spark。
   
   ![Hive运行机制图](..\img\Hive运行机制图.png)
   
   Hive通过给用户提供一系列交互接口，接受到用户的指令（SQL），使用自己的Driver，结合元数据（Metastore)，将这些指令翻译成MapReduce，提交到Hadoop中执行，最后，将执行返回的结果输出到用户交互接口。

## 1.4Hive 和数据库比较

由于Hive采用了类似SQL的查询语言HQL（Hive Query Language），因此很容易将Hive理解为数据库。其实从结构上来看，Hive和数据库除了拥有相似的查询语言，再无类似之处。本文将从多个方面来阐述Hive和数据库的差异。数据库可以用来Online的应用中，但是Hive是为数据仓库而设计的，清楚这一点，有利于从应用角度理解Hive的特性。

### 1.4.1 查询语言

由于SQL被广泛应用在数据仓库中，因此，专门针对Hive的特性设计了类SQL的查询语言HQL。熟悉SQL开发的开发这可以很方便的使用Hive进行开发。

### 1.4.2 数据更新

由于Hive是针对数据仓库应用设计的，而<font color=red>数据仓库的内容是读多写少的</font>。因此，<font color=red>Hive中不建议对数据的改写，所有的数据都是加载的时候确定好的</font>。而数据库中的数据通常是需要经常进行修改的。因此可以使用INSERT INTO ... VALUES...添加数据，使用UPDATE...SET修改数据。

### 1.4.3 执行延迟

Hive在查询数据的时候，由于没有索引，需要扫描整个表，因此延迟较高。另外一个导致Hive执行延迟高的因素是MapReduce框架。由于MapReduce本身具有较高的延迟，因此再利用MapReduce执行Hive查询时，也会有较高的延迟。相对的，数据库的执行延迟较低。当然，这个低是有条件的，即数据规模较小，当数据规模达到查过数据库的处理能力的时候，Hive的并行计算显然能体现出优势。

### 1.4.4 数据规模

由于Hive建立在集群上并可以利用MapReduce进行并行计算，因此可以支持很大规模的数据；对应的，数据库可以支持的数据规模较小。

|            | 查询语言 | 数据操作         | 执行延迟                                       | 数据规模                   |
| ---------- | -------- | ---------------- | ---------------------------------------------- | -------------------------- |
| Hive       | HQL      | 读多写少         | 高，因素：MapReduce本身具有延迟；加上没有索引  | 大规模集群数据             |
| 一般数据库 | SQL      | 通常需要修改数据 | 规模较小时，执行效率高，规模越大，执行效率越低 | 单机数据或其他小规模数据体 |

> 本章小Tip：Hive引擎有mr、spark、tez

# 第二章 Hive安装

## 2.1 Hive安装地址

1. Hive官网地址：<http://hive.apache.org/>
2. 文档查看地址：<https://cwiki.apache.org/confluence/display/Hive/GettingStarted>
3. 下载地址：<http://archive.apache.org/dist/hive>
4. github地址：<https://github.com/apache/hive>



## 2.2 Hive安装部署

### 2.2.1 安装Hive

