---
layout: post
title: Postgresql query optimization practice
key: 20170811
tags: sql optimization postgresql
---

Recently, I was develop a RESTful API for all business data in Australia. There are some thoughts I'd like to mark down for notes.

### Guides

* Denormalize if necessary
* Understand sql query order 
* Small table join big table

### Denormalize 

When designing models, we tend to normalize the database into a very detailed level because it makes sense or avoids duplication. However, how to normalize the database must depends on how the query is structured. 

### SQL query order

As far as I am concerned, SQL query order is essential to optimize a sql query. 

```sql

(8)SELECT (9)DISTINCT  (11)<Top Num> <select list>
(1)FROM [left_table]
(3)<join_type> JOIN <right_table>
(2)ON <join_condition>
(4)WHERE <where_condition>
(5)GROUP BY <group_by_list>
(6)WITH <CUBE | RollUP>
(7)HAVING <having_condition>
(10)ORDER BY <order_by_list>

```

To begin with, every step will produce a virutal table[^1].

#### FROM clause 

The from clause is the first step of sql query. 

* form t1

Will use t1 as vt1 

* from t1, t2, ..t

Will [cartesian product](https://en.wikipedia.org/wiki/Cartesian_product) t1 to tn and produce vt1. **Do not do this**

* from (nest sql query)

Will use nest sql query as result for vt1.

#### ON clause
Based on the on clause, filter vt1 as vt2. On clause need to work with join clause.

#### JOIN clause

Add out fields on vt2 in terms of on clause and produce vt3

#### WHERE clause

Filter the vt3 according to where clause and produce vt4

#### GROUP BY clause

Group vt5 by fields in the cause and produce vt5

#### WITH clause

Will give a sub-query block a name and add to vt5 > vt6

#### HAVING clause

Work with GROUP BY filter filter vt6 > vt7

#### SELECT clause

Select fields from vt7 > vt8

#### DISTINCT clause

Will remove duplicate on vt8 based on fields from distinct clause > vt9

#### ORDER BY clause

Will sort vt9 according to order by clause > vt10

#### TOP Clause

Will return vt10 with certain number > vt11


### Small table join big table

According to my view, this is ultimate goal of optimization the sql query.



### Reference 

[^1]: Inside Microsoft SQL Server 2005, **Logical Query Processing**, http://www.polyteknisk.dk/related_materials/9780735623132_chapter_01.pdf 




