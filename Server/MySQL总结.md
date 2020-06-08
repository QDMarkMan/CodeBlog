# MySQL总结

[toc]

以下是在学习`MySQL`的过程中的一些**基础但是重要**的记录。

## 表相关

- 查看表结构时的列属性

  ```bash
  DESC table;
  // 示例
  DESC student_info;
  +-----------------+-------------------+------+-----+---------+-------+
  | Field           | Type              | Null | Key | Default | Extra |
  +-----------------+-------------------+------+-----+---------+-------+
  | number          | int(11)           | NO   | PRI | NULL    |       |
  | name            | varchar(5)        | YES  |     | NULL    |       |
  | sex             | enum('男','女')   | YES  |     | NULL    |       |
  | id_number       | char(18)          | YES  | UNI | NULL    |       |
  | department      | varchar(30)       | YES  |     | NULL    |       |
  | major           | varchar(30)       | YES  |     | NULL    |       |
  | enrollment_time | date              | YES  |     | NULL    |       |
  +-----------------+-------------------+------+-----+---------+-------+
  ```

  列解释

  1. `Null`： 表示这个列是否可以存储`Null`值。
  2. `Key`: 表示当前列的键的信息， `PRI`是主键，`UNI`是`UNIQUE KEY`。
  3. `Default`:  表示默认值。
  4. `Extra`: 表示额外信息。

  

- 在插入数据时可以指定部分列。

  ```mysql
  -- 插入一行first_column值为2的数据， 其他的数据一般都为null
  INSERT INTO first_table(first_column) VALUES(2);
  
  ```

- 主键

  有时候我们可以通过某个列或者某些列确定唯一的一条记录，我们把这些列成为`候选键`。**一个表可能有多个候选键，我们可以选择一个作为表的主键，主键的值不能重复，通过主键可以找到唯一的一条记录**。我们通过以下的方式来指定主键。主键默认是有`NOT NULL`属性的。

  ```mysql
  -- 1:单列：直接在列的后面声明PRIMARY KEY
  CREATE TABLE student_info (
      number INT PRIMARY KEY,
      name VARCHAR(5),
      sex ENUM('男', '女'),
      id_number CHAR(18),
      department VARCHAR(30),
      major VARCHAR(30),
      enrollment_time DATE
  );
  -- 2: 多列提出来单独声明
  CREATE TABLE student_score (
      number INT,
      subject VARCHAR(30),
      score TINYINT,
      PRIMARY KEY (number, subject)
  );
  ```

  **注： 如果我们在创建表的时候声明了主键，那么MySQL会对我们插入的记录做校验， 如果新插入的主键重复了，就会报错。**

  <font color="red">主键和UNIQUE约束的区别</font>

  1. 一张表只能定义一个主键，UNIQUE约束可以有多个。
  2. 主键不允许存放`NULL`, 而声明了 `UNIQUE`属性的列可以存放`NULL`，而且可以重复的出现在多条记录中。

- 外键

  如果我们有一个学生的成绩表中的学号number列的值无法在我们的学生信息表的number列找到，就相当于插入了成绩但是我们不知道这是谁的成绩。`MySQL`为我们提供了外键约束机制来解决这种问题。语法如下

  ```mysql
  -- 如果A表中的某个列依赖B表中的某一个或者多个列，那A表就是子表，B表是父表， 通过外键来关联起来这两个表。
  CONSTRAINT [外键名称] FOREIGN KEY(列1, 列2, ...) REFERENCES 父表名(父列1, 父列2, ...);
  
  -- 例如
  CREATE TABLE student_score (
      number INT,
      subject VARCHAR(30),
      score TINYINT,
      PRIMARY KEY (number, subject),
      CONSTRAINT FOREIGN KEY(number) REFERENCES student_info(number)
  );
  -- 这样在对student_score表插入数据的时候， MySQL会替我们检查number是否存在于student_info表中， 不存在的话就会报错。
  ```

- `AUTO_INCREMENT`

  自增属性，整数类型或者浮点数类型的列可以设置自增属性。

  <font color="red">注意点:</font>

  1. 一个表中只能有一个列有该属性。
  2. 具有`AUTO_INCREMENT`的列必须建立索引。
  3. 拥有`AUTO_INCREMENT`属性的列不能再指定默认值。
  4. 一般拥有`AUTO_INCREMENT`属性的列都是作为主键的属性，来自动生成唯一标识一条记录的主键值。

## 查询

- 别名只是在本次查询的到的结果集中展示，而不会改变真实表中的列名。下一次查询中你对某一列取其他的别名也可以。

  ```mysql
  SELECT [列名] AS [别名] FROM [表名];
  ```

- 在查询多个列的时候， 列可以按照任意顺序排列，结果集将会按照我们指定的列名顺序显示， 比如我们查询`student_info`中的多个列。

  ```bash
  SELECT number, name, id_number, major FROM student_info;
  +----------+-----------+--------------------+--------------------------+
  | number   | name      | id_number          | major                    |
  +----------+-----------+--------------------+--------------------------+
  | 20180101 | 杜子腾    | 158177199901044792 | 计算机科学与工程         |
  | 20180102 | 杜琦燕    | 151008199801178529 | 计算机科学与工程         |
  +----------+-----------+--------------------+--------------------------+
  ```

- 在进行查询的时候，除非你确实需要表中的全部列，否则一般不要用*来查询所有，虽然这个方法很方便。但是查询不需要的列确实会**降低性能**。

- 查询去重使用`DISTINCT`关键字，但是如果查询多列，<font color="red">去重需要满足两条记录每一列的值都相同</font>。

  ```mysql
  SELECT DISTINCT 列名 FROM 表名;
  ```

- 限制条数（分页）查询。

  ```mysql
  LIMIT 开始行 限制条数;
  ```

- 结果排序，在`MySQL`中如果我们想要查询结果中的记录按照某种特定的规则排序，那我们必须显式的指定排序规则, <font color="red">`ORDR BY `语句必须放在 `LIMIT`语句前面</font>

  ```mysql
  -- ASC 升序（从小到大）  默认值
  -- DESC 降序（从大到小）
  
  -- 单列排序
  SELECT * FROM 表名 ORDER BY 列名 DESC;
  -- 多列排序
  SELECT * FROM 表名 ORDER BY 列1 ASC|DESC, 列2 ASC|DESC ...
  ```

- 查询`NULL`:  不能用普通操作符来与`Null`值进行比较，必须用`IS NULL`或者`IS NOT NULL`。

  ```mysql
  // 示例
  SELECT name from student WHERE name IS NOT NULL;
  ```

- `AND/OR`：在进行复杂的组合查询的时候，<font color="red">`AND`操作符的优先级会高于`OR`, 也就是在说对条件进行判断的时候的时候会先检测`AND`操作符两边的内容。</font>例如下面的情况。

  ```mysql
  -- bad
  number > 95 OR number < 55 AND name = '测试AND';
  -- good
  (number > 95 OR number < 55) AND name = '测试AND';
  ```

  上面的这条语句 可以看作`number` < 95  或者 `number < 55 AND name = '测试AND'`任意一条成立。为了避免这种尴尬的情况出现， **建议加上小括号来显性的指定各个搜索的检测条件**。

- 通配符： <font color=red>`LIKE`和`NOT LIKE`操作符只用于字符串匹配，通配符不能代表`NULL`。</font>

  `%`:  表示任意一个**字符串**。 `张%`： 以张开头的字符串,  `%张%`： 包含张的字符串。

  `-`：表示任意一个**字符**。`%`匹配范围太大，这个时候我们就使用这个。

  如果字符串中本身就包含`%, _`字符， 那这个时候要匹配他们就要用`\%,\_`进行转义。
  
- 聚集（统计）函数

  - `COUNT(column/*)`:  返回某列的数量。
  - `MAX(column/*)`: 返回某列的最大值。
  - `MIN(column/*)`: 返回某列的最小值。
  - `SUM(column/*)`: 返回某列的值之和。
  - `AVG(column/*)`: 返回某列的平均数。

  如果存在搜索条件，那么<font color="red">不在搜索条件中的那些记录是不参与统计的。</font>

  另外：<font color="red">聚集函数不能放到WHERE子句中！！！</font>

- 分组查询： 针对某个列，将该列相同值的记录分到一个组中，主要是方便我们进行信息的统计。我们在使用分组的时候要注意：<font color="red">分组的存在仅仅是为了方便我们分别统计各个组中的信息，所有我们只要把分组列和统计函数放到查询列表处就好了。</font>

  如果我们想对各个分组查询出来的统计数据进行排序，需要为查询列表中有聚集函数的表达式添加`别名`。

  注意事项：

  1. 如果分组列中含有`NULL`, 那么它会作为一个单独的分组存在。
  2. 如果存在多个分组列，那么分组的作用将会作用在最后的那个分组列上。
  3. `GROUP BY`子句必须出现在`WHERE`子句之后，`ORDER BY`子句之前， 而且`ORDER BY`语句要在`HAVING`语句之后。
  4. `非分组列`不能单独出现在检索列表中(可以被放到聚集函数中)。
  5. `GROUP BY`子句后也可以跟随`表达式`(但不能是聚集函数)。
  6. `WHERE`语句作用于分组之前，作用于每一条语句。但是`HAVING`语句作用于分组之后，作用于整个分组。

- 简单查询语句中各个子句的顺序。<font color="red">[]表示可以省略，书写的时候必须严格按照这个顺序， 否则会报错</font>

  ```mysql
  SELECT [DISTINCT] 查询列表
  [FROM 表名]
  [WHERE 条件]
  [GROUP BY 分组列]
  [HAVING 分组过滤条件]
  [ORDER BY 排序列表]
  [LIMIT 开始行，总条数]
  ```

- 子查询：子查询也称为`内层查询`，一般是使用内层查询的结果作为外层查询的条件，然后按照由内而外的顺序执行查询语句。主要是满足多表查询的目的。

  子查询分为以下几种

  **不相关子查询**：子查询和外层查询都没有依赖关系。

  1. 标量子查询：查询的结果是单纯的一个值。
  2. 列子查询：查询的值是多个，对应一列。
  3. 行子查询：查询一整行的数据，需要用`LIMIT 1`限制。
  4. 表子查询：查询结果是多行多列的数据。
  5. `EXISTS`/`NOT EXISTS`查询：不关心查询结果时使用。

  **相关子查询**：在子查询的语句中引用到外层查询的值，例如：

  ```mysql
  -- 相关子查询
  SELECT *
  FROM student_info
  WHERE EXISTS (SELECT * FROM student_score WHERE student_score.number = student_info.number);
  
  -- 查询过程:
  -- 先执行外层查询获得到student_info表的第一条记录，发现它的number值是20180101。把20180101当作参数传入到子查询，此时子查询的意思是判断student_score表的number字段是否有20180101这个值存在，子查询的结果是该值存在，所以整个EXISTS表达式的值为TRUE，那么student_info表的第一条记录可以被加入到结果集。
  -- 再执行外层查询获得到student_info表的第二条记录，发现它的number值是20180102，与上边的步骤相同，student_info表的第二条记录也可以被加入到结果集。
  -- ...
  -- 再执行外层查询获得到student_info表的第五条记录，发现它的number值是20180105，把20180105当作参数传入到它的子查询，此时子查询的意思是判断student_score表的number字段是否有20180105这个值存在，子查询的结果是该值不存在，所以整个EXISTS表达式的值为FALSE，那么student_info表的第五条记录就不被加入结果集中。
  ```

  注：

  <font color="red">所有的子查询都要用()扩起来。</font>

  <font color="red">子查询虽然涉及多个表，但是最终产生的结果还是用来展示外层查询的结果，子查询的结果只是当作中间结果来使用。</font>

- 连接查询：<font color="red">是一种使用一个查询语句结果集中展示多个表的信息的方式</font>。

  为了解决<font color="red">驱动表中的记录即使在被驱动表中没有匹配的记录，也仍然需要加入到结果集</font>的问题。出现了内/外连接的概念。
  
  **内连接**：**驱动表中记录在被驱动表中找不到匹配的记录**，该记录不会加入到最后的结果集。<font color="red">内连接中的WHERE子句和ON子句是等价的。</font> 语法如下：
  
  ```mysql
  SELECT * FROM t1 [INNER | CROSS] JOIN t2 [ON 连接条件] [WHERE 普通过滤条件];
  -- 也就是说下面的写法都是等价的
  SELECT * FROM t1 JOIN t2;
  SELECT * FROM t1 INNER JOIN t2; -- 推荐写法 因为语意最明确
  SELECT * FROM t1 CROSS JOIN t2;
  SELECT * FROM t1, t2;
  ```
  
  **外连接**：驱动表中的记录即使在被驱动表中没有匹配的记录，也仍然需要加入到结果集。对于驱动表的选取不同也有两种不同的方式：
  
  - 左外连接：选取**左侧**的表作为驱动表。
  - 右外连接：选取**右边**的表作为驱动表。
  
  需要注意的是，这个`ON`子句是专门为外连接驱动表中的记录在被驱动表找不到匹配记录时应不应该把该记录加入结果集这个场景下提出的。
  
  **左（外）连接的语法**：
  
  ```mysql
  SELECT * FROM t1 LEFT [OUTER] JOIN t2 ON 连接条件 [WHERE 普通过滤条件];
  ```
  
  内链接和外连接的**根本区别**是<font color="red">在驱动表中的记录不符合`ON`子句中的连接条件时会不会把记录加入到最后的结果集中。</font>
  
  **ON/WHERE**：写在ON里面的条件即使不满足也会返回该行，只是对应的列为NULL； 写在WHERE里面的条件则不满足就不返回。
  
  多表连接就是把多个表连接到一起， 语法如下
  
  ```mysql
  -- 多表连接
  SELECT * FROM t1 INNER JOIN t2 INNER JOIN t3
  WHERE t1.m1 = t2.m2 AND t1.m1 = t3.m3;
  ```
  
- 组合（合并）查询：用来将多条查询语句产生的结果集合并起来。 

  ```sql
  -- 正常的方式
  SELECT m1 FROM t1 WHERE m1 < 2 OR m1 > 2;
  -- 组合查询的方式
  SELECT m1 FROM t1 WHERE m1 < 2 UNION SELECT m1 FROM t1 WHERE m1 > 2;
  -- 更加推荐的方式
  (SELECT m1 FROM t1 WHERE m1 < 2) UNION (SELECT m1 FROM t1 WHERE m1 > 2) ORDER BY m1 LIMIT 1;
  ```

  `UNION`连接的<font color="red">各个语句查询的数量是要相同的</font>，而且最好各个查询列表中位置相同的表达式的类型也是相同的。

  组合查询的结果集中显示的列名将以第一个查询的列为准。

  `UNION` 会主动的进行记录的去重，如不想去重复的话，可以是用`UNION ALL`。

  组合查询的排序和限制等条件可以跟在最后面，但是如果单独对一个查询语句进行排序，是没有用的。因为<font color="red">组合查询并不保证最后汇总起来的大结果集中的数据是按照各个小查询的结果中的顺序排序 </font>。

- 各种查询应用场景的区别

  分组查询：应用于统计。

  子查询：适用于需要一个或者多个查询值为查询条件的多表查询。

  连接查询：适用于使用一个表展示多个表信息的情况。

## 数据插入/编辑/删除

- 数据插入，语法：

  ```sql
  INSERT INTO 表名 VALUES(列1的值，列2的值, ..., 列n的值);
  -- 指定顺序
  INSERT INTO first_table(first_column, second_column) VALUES (3, 'ccc');
  -- 批量插入
  INSERT INTO 表名 VALUES(列1的值，列2的值, ..., 列n的值), (列1的值，列2的值, ..., 列n的值);
  ```

  <font color="red">`VALUES`语句必须给出表中所有列的值，少一个都不行。也就是说INSERT语句中指定的列顺序可以改变，但是一定要和`VALUES`列表中的值一一对应起来</font>

  如果想插入部分值，那么被忽略的列一定要满足下面两个条件：

  1. 该列允许存储`NULL`值。
  2. 该列有`DEFAULT`值，给出了默认值。

  `INSERT IGNORE`用于对于那些是主健或者具有`UNIQUE`约束的的列或者列组合来说，如果表中已存在的记录中没有与待插入记录在这些列或者列组合上重复的值，那么就把待插入的记录插入到表中，否则忽略此次操作。