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