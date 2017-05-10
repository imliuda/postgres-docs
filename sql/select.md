# SELECT

Name

SELECT, TABLE, WITH -- retrieve rows from a table or view

## 摘要

```
[ WITH [ RECURSIVE ] with_query [, ...] ]
SELECT [ ALL | DISTINCT [ ON ( expression [, ...] ) ] ]
    [ * | expression [ [ AS ] output_name ] [, ...] ]
    [ FROM from_item [, ...] ]
    [ WHERE condition ]
    [ GROUP BY grouping_element [, ...] ]
    [ HAVING condition [, ...] ]
    [ WINDOW window_name AS ( window_definition ) [, ...] ]
    [ { UNION | INTERSECT | EXCEPT } [ ALL | DISTINCT ] select ]
    [ ORDER BY expression [ ASC | DESC | USING operator ] [ NULLS { FIRST | LAST } ] [, ...] ]
    [ LIMIT { count | ALL } ]
    [ OFFSET start [ ROW | ROWS ] ]
    [ FETCH { FIRST | NEXT } [ count ] { ROW | ROWS } ONLY ]
    [ FOR { UPDATE | NO KEY UPDATE | SHARE | KEY SHARE } [ OF table_name [, ...] ] [ NOWAIT | SKIP LOCKED ] [...] ]

where from_item can be one of:

    [ ONLY ] table_name [ * ] [ [ AS ] alias [ ( column_alias [, ...] ) ] ]
                [ TABLESAMPLE sampling_method ( argument [, ...] ) [ REPEATABLE ( seed ) ] ]
    [ LATERAL ] ( select ) [ AS ] alias [ ( column_alias [, ...] ) ]
    with_query_name [ [ AS ] alias [ ( column_alias [, ...] ) ] ]
    [ LATERAL ] function_name ( [ argument [, ...] ] )
                [ WITH ORDINALITY ] [ [ AS ] alias [ ( column_alias [, ...] ) ] ]
    [ LATERAL ] function_name ( [ argument [, ...] ] ) [ AS ] alias ( column_definition [, ...] )
    [ LATERAL ] function_name ( [ argument [, ...] ] ) AS ( column_definition [, ...] )
    [ LATERAL ] ROWS FROM( function_name ( [ argument [, ...] ] ) [ AS ( column_definition [, ...] ) ] [, ...] )
                [ WITH ORDINALITY ] [ [ AS ] alias [ ( column_alias [, ...] ) ] ]
    from_item [ NATURAL ] join_type from_item [ ON join_condition | USING ( join_column [, ...] ) ]

and grouping_element can be one of:

    ( )
    expression
    ( expression [, ...] )
    ROLLUP ( { expression | ( expression [, ...] ) } [, ...] )
    CUBE ( { expression | ( expression [, ...] ) } [, ...] )
    GROUPING SETS ( grouping_element [, ...] )

and with_query is:

    with_query_name [ ( column_name [, ...] ) ] AS ( select | values | insert | update | delete )

TABLE [ ONLY ] table_name [ * ]
```

## 描述

SELECT用于从0个或多个表中检索数据。SELECT语句的处理流程如下：

 1. 计算WITH语句中的查询。结果将会生成一个临时表，稍后可以在FROM语句中引用这张表，该表可以被多次引用，但只会计算一次，因此能够提升查询效率。

 2. 计算所有FROM列表中的所有元素（FROM列表中的元素或为一个真实的表，或者为一个虚拟表）。如果FROM列表中的元素数量大于1个，那么这些表将会采用cross-joined方式进行连接。（参考FROM从句）

 3. 如果存在WHERE从句，那么所有不满则条件的数据行会在输出结果中去除。（参考WHERE从句）

 4. 如果存在GROUP BY从句，或者存在聚合函数，输出结果将会按行进行分组，每行数据对应1个或多个组名，然后聚合函数进行计算。如果有HAVING从句，那么不满足条件的组会在输出结果中去除。（参考GROUP BY从句和HAVING从句。）

 5. 采用SELECT输出表达式进行计算，得到实际输出的数据行。

 6. SELECT DISTINCT去除结果中的重复行。SELECT DISTINCT ON去除结果中所有满足条件的数据行。SELECT ALL（默认）返回所有的数据行。

 7. 通过UNION，INTERSECT和EXCEPT，可以将多个SELECT语句的结果集合并成一个结果集。UNION返回在其中一个或同时在两个结果集中存在的行。INTERSECT返回两个结果集的交集，及同时在两个结果集中都存在。EXCEPT返回在第一个结果集中存在，但在第一个结果集中不存在的行。在上面这3中情况中，重复的数据行将会被移除，除非指定了ALL。通过指定DISTINCT明确说明重复的数据行要被移除。尽管在SELECT中默认采用ALL，但在这里DISTINCT是默认的方式。（参考UNION从句，INTERSECT从句和EXCEPT从句。）

 8. 如果存在ORDER BY从句，返回的数据行会以指定的方式进行排序。如果没有指定ORDER BY从句，返回的数据行的顺序可能以任何系统最快查找来生成这些数据的方式排序。

 9. 如果LIMIT（或FETCH FIRST）或OFFSET从句存在，则只返回结果集的子集。

 10. 如果FOR UPDATE，FOR NO KEY UPDATE，FOR SHARE或者FOR KEY SHARE存在，SELECT将会锁定这些选中的数据行以防止同时更新。

你必须对SELECT列表中的每一列都拥有SELECT权限。要使用FOR NO KEY UPDATE，FOR UPDATE，FOR SHARE或者FOR KEY SHARE，也需要拥有UPDATE权限（for at least one column of each table so selected）。

## 参数

### WITH从句

WITH从句允许你指定一个或多个子查询，在主查询中可以通过名称来引用这些子查询。在主查询语句执行的过程中，这些字查询作为临时的表或视图存在。每个子查询可以是SELECT，TABLE，VALUES，INSERT，UPDATE或者DELETE语句。当WITH从句是修改数据的语句，通常会着一个RETURNING从句，返回的是RETURNING的返回，并不是WITH从句修改的这个表的返回。如果省略了RETURNING，语句仍然会执行，但不会产生任何输出，所以在主查询中不能引用这个临时表。

对于每个WITH从句，必须要指定一个名称（非schema限定的表）。然后可以指定一些列可选的列名称，如果省略，列名则由子查询自动推断。

如果指定了RECURSIVE，则允许SELECT子查询通过名称来引用自己，这样的子查询必须是下面的这种形式：

```
non_recursive_term UNION [ ALL | DISTINCT ] recursive_term
```

对自身的递归引用必须位于UNION的右侧，每个子查询只能对自身进行一次递归引用，不支持递归修改数据，但是可以在修改语句中使用递归子查询的结果。参考7.8节中的示例。

RECURSIVE的另一个影响是，WITH子查询语句不必按顺序书写：一个子查询语句可以引用该语句后面的子查询（但是，环形引用或互相引用是为实现的）。若未指定RECURSIVE，则WITH子查询语句只能引用该语句之前的子查询。

WITH查询的特性在于，在主查询执行的过程中，他们只进行一次求值，即使主查询中多次引用了子查询。特别的是，对于修改数据的子查询，能够保证他们只执行一次。不管主查询中读取了全部或任何子查询的输出。

主查询和WITH查询是同时执的（概念上的）。WITH查询中修改数据语句产生的效果对于查询中的其他部分是不可见的。而不是读取读取RETURNING的输出。如果两个子查询同时修改同一行数据，结果时不可预料的。

参考7.8节获取额外信息。

### FROM从句

FROM为SELECT指定一个和多个表，如果指定了多个表，则结果是所有表的卡迪尔积（cross join）。通常，会加上一些限定条件（通过where）来获取卡迪尔积中一小部分子集。

FROM从句可以包含下列元素：

***table_name***

一个已经存在的数据表或试图的名称（可通过schema限定）。如果表名前指定了ONLY，则只有这个表会被扫描，如果未指定ONLY，则该表和所有的派生表（如果有）会被扫描。可选的，如果表名后面指定了\*，则明确指示包含所有的派生表。

***alias***

FROM元素的替代名称。别名用来保持语句的简洁或者消除自连接（self-join，同一个表会被多次扫描）的歧义。通过别名，可以完全隐藏表或函数的真是名称。例如 FROM foo AS f，SELECT语句的其他部分必须通过f来引用这个FROM元素，而不是foo。如果指定了别名，还可以指定一个列的别名列表，作为表的列名的替换。

TABLESAMPLE ***sampling_method*** ( ***argument*** [, ...] ) [ REPEATABLE ( ***seed*** ) ]

表名后的TABLESAMPLE从句用来指示，要采用指定的sampling_method获取表中数据行的一个子集。抽样会在任何其他过滤器之前进行，例如WHERE。标准的PostgreSQL发行版提供了两个抽样方法BERNOULLI和SYSTEM，其他的抽样方法可以通过数据库的扩展进行安装。

BERNOULLI和SYSTEM抽样方法都接受一个argument参数，是一个小数表示的对标进行的采样率，表示为0-100之间的百分比，该参数可以是任何实数类型的表达式（别的抽样方法可能接受多个或不同参数）。这两个方法返回随机选取的样本，样本的数量和总数的比值近似于参数中指定的百分比。BERNOULLI方法对整个表进行扫描，表的数据行之间是独立的，每一行均具有相同的选择或忽略的概率。SYSTEM方法采用块级别的抽样方式，每个块具有特定级别的概率，块中的所有数据行都会被返回。当指定的采样比例比较小时，SYSTEM方法明显要比BERNOULLI方法块，但是由于集群的影响，它返回的样本数据的随机性会差一些。

可选的REPEATABLE从句指定一个种子数字，用来在采样方法中生成随机数，该数可以为任何非空的浮点型值。对于两个采用了相同的argument和seed参数的查询，他们会返回相同的样本，如果期间表未被修改的话。但是不同的种子通常会产生不同的样本。如果未指定REPEATABLE，那么对于每个查询，都会基于系统生成一个种子，重新生成一个随机的样本。提示，有些插件提供的抽样方法不支持REPEATABLE，并且每次总是能返回新的样本。

***select***

FROM从句中可以包含子SELECT语句。表现行为是，就像是在SELECT的执行过程中，通过子查询的输出创建了一个临时的表。注意，子SELECT语句必须用括号括起来，并且要为之提供一个别名。也可以在这里使用VALUE指令。

***with_query_name***

通过名称来引用WITH子查询，就如同是一个表名一样（实际上，考虑到主查询，WITH子查询故意隐藏了真实的表名。如果需要，你可以通过使用schema限定的方式来引用真实的表名）。

***function_name***

FROM从句中可以使用函数调用（可以使用任何函数，对于能够返回结果的函数，尤其有用）。就像是在SELECT执行过程中，通过函数的输出创建了一个临时的表。当指定了WITH ORDINALITY从句，会在所有函数输出列的后面添加一个数字序号列。

可以像表一样为其指定一个别名。当指定了别名，还可以指定一个列的别名列表，用来替换函数返回的符合类型数据中一个或多个的属性名，包括ORDINALITY添加的列。

通过使用ROWS FROM( ... )，可以将多个函数结合起来，作为一个单独的FROM元素。该元素的输出是每个函数的第一行数据的连接，第二行数据的连接（concatenation），依次类推。如果一些函数产生的行的数量小于其他函数的，将会采用空值替替换缺失的数据，所以返回的行的数量总是等于函数所生成的最大行数。

如果函数定义为范围*record*数据类型，那么必须要指定别名（AS关键字），紧跟如下形式的列定义列表( column_name data_type [, ... ])。列定义列表必须和函数返回的实际列的数量和类型相匹配。

当采用ROWS FROM( ... )时，如果其中一个函数需要列定义列表，那么最好将列定义列表放在ROWS FROM( ... )中的函数调用之后。只有当只有一个函数，并且没有WITH ORDINALITY从句时，才可以将列定义列表放在ROWS FROM( ... )之后。

要同时使用列定义列表和ORDINALITY，必须采用ROWS FROM( ... )的方式，然后再ROWS FROM( ... )中进行列列表定义。

***join_type***

下列其中之一

 - [ INNER ] JOIN
 - LEFT [ OUTER ] JOIN
 - RIGHT [ OUTER ] JOIN
 - FULL [ OUTER ] JOIN
 - CROSS JOIN

对于INNER和OUTER连接类型，必须指定连接条件，且必须是NATURAL，ON join_condition或者USING (join_column [, ...])其中之一。解释参考下面，对于 CROSS JOIN，则不允许出现这些从句。

JOIN从句连接两个FROM元素，为简明起见，我们先把FROM元素叫做表，尽管，他们可能为任一类型的FROM元素。必要时，使用括号来决定嵌套顺序。
