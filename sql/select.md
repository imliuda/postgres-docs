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

WITH从句允许你指定一个或多个子查询，在主查询中可以通过名称来引用这些子查询。在主查询语句执行的过程中，这些字查询作为临时的表或视图存在。每个子查询可以是SELECT，TABLE，VALUES，INSERT，UPDATE或者DELETE语句。当WITH从句是修改数据的语句，通常会着一个RETURNING从句。
