# MySQL总结

[toc]

以下是在学习`MySQL`的过程中的一些重点总结项目。

## 表

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

  ```mssql
  -- 插入一行first_column值为2的数据， 其他的数据一般都为null
  INSERT INTO first_table(first_column) VALUES(2);
  
  ```

- 主键

  有时候我们可以通过某个列或者某些列确定唯一的一条记录，我们把这些列成为`候选键`。**一个表可能有多个候选键，我们可以选择一个作为表的主键，主键的值不能重复，通过主键可以找到唯一的一条记录**。我们通过以下的方式来指定主键。主键默认是有`NOT NULL`属性的。

  ```mssql
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

- 