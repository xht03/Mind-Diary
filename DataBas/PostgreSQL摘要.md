---
title: PostgreSQL摘要
date: 2024-04-01 13:34:43
tags:
- original
categories:
- The Introduction to Database
---

 Relational Databases 关系数据库

Database Design 

Data Storage and Query

Transaction Management



### Relational Model

#### Structure 结构

> 在 *关系模型* 中：每一个表是一个关系，每一行是一个元组，每一列是一个属性。

- A relational database consists of a collection of tables.
- A row in a table represents a relationship among a set of values.
- The order in which tuples appear in a relation is irrelevant, since a relation is a set of tuples .



#### Domain 域

> 域就是属性的值域（取值集合）。原子性意为“**不可再分**”。

- For each attribute of a relation, there is a set of permitted values , called the domain of that attribute.

- For all relations r , the domains of all attributes of *r* be **atomic**.



 #### Null Value 空值

- 空值是一种特殊值，表示：取值未知或不存在。
- 任何域都含有空值。



#### Schema 模式

Database schema : the logical design of the database.

Database instance : a snapshot of the data in the database at a given instant in time.

> 例如：`instructor = (ID, name, dept_name, salary)`



#### Keys 键

> We must have a way to specify how tuples within a given relation are distinguished.

- **superkey** : A superkey is a set of one or more attributes that, taken collectively, allow us to identify uniquely a tuple in the relation.
- **candidate key** : The superkeys, for which no proper subset is a superkey, are called candidate keys.
- **primary key** : A primary key is a candidate key that is chosen by the database designer as the principal means of identifying tuples within a relation.
- **foreign key** : A relation, may include among its attributes the primary key of another relation. This attribute is called a foreign key.

> 注意：
>
> 1. A superkey may contain extraneous attributes.
>
> 2. It is possible that several distinct sets of attributes could
>    serve as a candidate key.



#### Schema Diagrams 模式图

> Schema diagram = database schema + primary key and foreign key dependencies

![Schema Diagrams](C:\Users\Keats\Desktop\Erewhon\Yanxu-Blog\source\_posts\PostgreSQL摘要\schema_diagrams.png)



#### Query Languages

A query language is a language in which a user requests information from the database.

> Query Languages 通常有两类：
>
> - *procedural*：指定查询什么数据以及如何获取数据。
> - *declarative*：只需指定查询什么数据，不必指定如何查询数据。









### Relational Algebra









### SQL

#### What is SQL

数据库系统提供：

- *data-definition language* (DDL)：定义数据库模式。
- *data-manipulation language* (DML)：表达数据库查询和更新。

但实际上，*DDL* 和 *DML* 并不是两种分离的语言。相反地，它们简单地构成单一的数据库语言，比如：SQL。

> 查询 (query) 是对所求信息进行检索的语句。DML中涉及信息检索的部分称作 *query language* 。但实践中常将 query language 和 DML 视为同义词。



#### Domain Types

- `char(n)`：固定长度字符串（长度为 n ）
- `varchar(n)`：变长度字符串（最大长度为 n ）
- `int`：整数
- `smallint`：小整数
- `numeric(p, d)`：固定点数（有效数字 p 位，小数点后 d 位）
- `real`、`double percision`：实数、双精度浮点数
- `float(n)`：浮点数（有效数字 n 位）



#### Common Operation

> 创建 table

```sql
CREATE TABLE Users(
	user_id char(6) NOT Null,
	username varchar(30),
	realname varchar(30),
	age int,
	password varchar(30),
	permisson int
);
```

> 创建 procedure

```

```



### Relational Database Design

#### 函数依赖

> 函数依赖本质上就是：两个属性之间存在单射

设R是一个关系模式，X 和 Y 是 R 的属性集的子集。如果对于 R 的任意一个关系r，对于r中的任意两个元组 t1 和 t2 ，只要 t1 和 t2 在 X 上的值相等，那么它们在 Y 上的值也相等，那么我们就说 Y 在 X 上函数依赖，记作 `X->Y` 。



K is a superkey for R &harr; `K -> R`

K is a candidate key for R &harr; `K->R` and no &alpha;K,   R







#### 第一范式

R 满足第一范式 &harr; R的所有属性的域都是原子的。

