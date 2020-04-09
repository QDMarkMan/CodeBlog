# MySQL总结

[toc]

以下是在学习`MySQL`的过程中的一些重点总结项目。

## 表

- 在新增时可以指定部分列。

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

## 查询

- 