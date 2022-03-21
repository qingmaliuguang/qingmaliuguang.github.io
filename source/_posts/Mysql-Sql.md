---
title: Mysql事务
date: 2021-10-26 19:36:09
tags: Mysql

---

- INSERT ... ON DUPLICATE KEY UPDATE

  > 相当于 先判断一条记录是否存在，存在则执行后边的Update语句，否则执行前边的insert语句。
  >
  > 参考：https://www.cnblogs.com/moxiaotao/p/9431808.html

- replace into tbl_name(col_name, …) values(…)

  > replace into 首先尝试插入数据到表中，如果发现表中已经有此行数据（根据主键或者唯一索引判断）则先删除此行数据，然后插入新的数据。 否则，直接插入新数据。
  >
  > 参考：https://blog.csdn.net/liu379702831/article/details/105819304/

- replace into与INSERT ... ON DUPLICATE KEY UPDATE的区别

  > replace into和on duplcate key update都是只有在primary key或者unique key冲突的时候才会执行。如果数据存在，replace into则会将原有数据删除，再进行插入操作，这样就会有一种情况，如果某些字段有默认值，但是replace into语句的字段不完整，则会设置成默认值。而on duplicate key update则是执行update后面的语句。 
  >
  > 参考：https://blog.csdn.net/qq2430/article/details/80511640