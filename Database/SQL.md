# SQL 学习笔记

[TOC]

> 基于 `MySQL 8`

## 数据库

### 创建数据库

```SQL
CREATE DATABASE `数据库名`;
```

```SQL
CREATE DATABASE `数据库名`
    DEFAULT CHARACTER SET = 'utf8mb4'
    DEFAULT COLLATE = 'utf8mb4_0900_ai_ci';
```

### 修改数据库

```SQL
ALTER DATABASE `数据库名`
    DEFAULT CHARACTER SET 'gb2312'
    DEFAULT COLLATE 'gb2312_chinese_ci';
```

### 删除数据库

```SQL
DROP DATABASE IF EXISTS `数据库名`;
```

### 查看数据库

```SQL
SHOW DATABASES;

SHOW DATABASES like '%pattern%';
```

## 数据表

### 创建数据表

```SQL
CREATE TABLE `customers`
(
    `id`      INT          NOT NULL AUTO_INCREMENT,
    `name`    VARCHAR(10)  NOT NULL,
    `address` VARCHAR(100) NULL,
    PRIMARY KEY (`id`)
);
```

### 修改表

#### 增加列

```SQL
ALTER TABLE `customers`
    ADD COLUMN `city` VARCHAR(10) NULL AFTER `name`;
```

#### 修改列

```
ALTER TABLE `customers`
    CHANGE COLUMN `name` `name` VARCHAR(5) NULL;
    
ALTER TABLE `customers`
    ALTER COLUMN `name` SET DEFAULT 'None';
    
ALTER TABLE `customers`
    MODIFY COLUMN `name` VARCHAR(10) NULL AFTER `city`;
```

#### 删除列

```SQL
ALTER TABLE `数据表名`
    DROP COLUMN `列名`;
```

#### 重命名表

```SQL
ALTER TABLE `customers` RENAME TO `customer`;

# 或

RENAME TABLE `customer` TO `customers`;
```

### 删除表

```SQL
DROP TABLE `数据表名`;
```

## 主键

### 删除主键

```SQL
ALTER TABLE `Product`
    DROP PRIMARY KEY;
```

### 添加主键

```SQL
ALTER TABLE `Product`
    ADD PRIMARY KEY (`product_id`);
```

## 连接

### 交叉连接

```SQL
SELECT *
    FROM `数据表1`
             CROSS JOIN `数据表2`;

# 或

# 省略 CROSS JOIN 改用逗号
SELECT *
    FROM `数据表1`, `数据表2`;
```

### 内连接

```SQL
SELECT *
    FROM `数据表1`
             INNER JOIN `数据表2` ON 连接条件;
             
# 省略 INNER 关键字
SELECT *
    FROM `数据表1`
             JOIN `数据表2` ON 连接条件;
```

### 外连接

#### 左外连接

```SQL
SELECT *
    FROM `数据表1`
             LEFT OUTER JOIN `数据表2` ON 连接条件;

SELECT *
    FROM `数据表1`
             LEFT JOIN `数据表2` ON 连接条件;
```

#### 右外连接

```SQL
SELECT *
    FROM `数据表1`
             RIGHT OUTER JOIN `数据表2` ON 连接条件;

SELECT *
    FROM `数据表1`
             RIGHT JOIN `数据表2` ON 连接条件;
```

## WHERE 子句

### [NOT] BETWEEN ... AND ...

```SQL
SELECT *
    FROM `Product`
    WHERE `sale_price` BETWEEN 1000 AND 5000;
```

```SQL
SELECT *
    FROM `Product`
    WHERE `sale_price` NOT BETWEEN 1000 AND 5000;
```

### IN

```SQL
SELECT *
    FROM `Product`
    WHERE `product_type` IN ('办公用品', '交通工具');
```

### IS [NOT] NULL

```SQL
SELECT *
    FROM `Product`
    WHERE `purchase_price` IS NULL;
```

```SQL
SELECT *
    FROM `Product`
    WHERE `purchase_price` IS NOT NULL;
```

### Limit

```SQL
SELECT *
    FROM `Product`
    ORDER BY `purchase_price` DESC
    LIMIT 0,3;
    
# 或

SELECT *
    FROM `Product`
    ORDER BY `purchase_price` DESC
    LIMIT 3 OFFSET 0;

# 或

SELECT *
    FROM `Product`
    ORDER BY `purchase_price` DESC
    LIMIT 3;
```

## CASE

### 例1

```SQL
SELECT `product_name`,
       CASE
           WHEN `product_type` = '衣服' THEN concat('A:', `product_type`)
           WHEN `product_type` = '办公用品' THEN concat('B:', `product_type`)
           WHEN `product_type` = '厨房用具' THEN concat('C:', `product_type`)
           ELSE NULL
           END AS `abc_product_type`
    FROM `Product`;

# 使用简单 CASE 表达式

SELECT `product_name`,
       CASE `product_type`
           WHEN '衣服' THEN concat('A:', `product_type`)
           WHEN '办公用品' THEN concat('B:', `product_type`)
           WHEN '厨房用具' THEN concat('C:', `product_type`)
           ELSE NULL
           END AS `abc_product_type`
    FROM `Product`;
```

输出

product_name | abc_product_type
--- | ---
T恤 | A:衣服
打孔器 | B:办公用品
运动T恤 | A:衣服
菜刀 | C:厨房用具
高压锅 | C:厨房用具
叉子 | C:厨房用具
擦菜板 | C:厨房用具
圆珠笔 | B:办公用品 

### 例2

```SQL
SELECT sum(CASE WHEN `product_type` = '衣服' THEN `sale_price` ELSE 0 END)   AS `sum_price_cloth`,
       sum(CASE WHEN `product_type` = '办公用品' THEN `sale_price` ELSE 0 END) AS `sum_price_kitchen`,
       sum(CASE WHEN `product_type` = '厨房用具' THEN `sale_price` ELSE 0 END) AS `sum_price_office`
    FROM `Product`;

# 使用 IF

SELECT sum(IF(`product_type` = '衣服', `sale_price`, 0))   AS `sum_price_cloth`,
       sum(IF(`product_type` = '办公用品', `sale_price`, 0)) AS `sum_price_kitchen`,
       sum(IF(`product_type` = '厨房用具', `sale_price`, 0)) AS `sum_price_office`
    FROM `Product`;
```

输出

sum_price_cloth | sum_price_kitchen | sum_price_office
--- | --- | ---
5002 | 602 | 11184

## 存储过程

### 例1

```SQL
# 创建
CREATE PROCEDURE `procName`()
BEGIN
    SELECT * FROM `Product`;
END;

# 调用
CALL `procName`;
```

### 例2

```SQL
# 创建
CREATE PROCEDURE `procName`(IN `id` CHAR(5))
BEGIN
    SELECT * FROM `Product` WHERE `product_id` = `id`;
END;

# 调用
CALL procName(`0001`);
```

## 函数

### 例1

```SQL
# 定义
CREATE FUNCTION funcName()
    RETURNS VARCHAR(100)
BEGIN
    RETURN (SELECT `product_name`
                FROM `Product`
                WHERE `product_id` = '0001');
END;

# 调用
SELECT funcName();
```

## 变量

```SQL
CREATE PROCEDURE `procName`()
BEGIN
    DECLARE `cus_product_id` VARCHAR(4) DEFAULT '0001';
    SET `cus_product_id` = '0002';

    SELECT `product_id`, `product_name` FROM `Product` WHERE `product_id` = `cus_product_id`;
END;

CALL `procName`();
```

```SQL
CREATE PROCEDURE `procName`()
BEGIN
    DECLARE `cus_product_id` VARCHAR(4);
    DECLARE `cus_prodcut_name` VARCHAR(100);

    SELECT `product_id`, `product_name`
        INTO `cus_product_id`, `cus_prodcut_name`
        FROM `Product`
        WHERE `product_id` = '0005';

    SELECT `cus_product_id`, `cus_prodcut_name`;
END;

CALL `procName`();
```

## 触发器

### 创建

#### 单语句触发器

```SQL
CREATE TABLE `account`
(
    `account_num` INT,
    `amount`   INT
);

CREATE TRIGGER `ins_sum`
    BEFORE INSERT
    ON `account`
    FOR EACH ROW SET @`sum` = @`sum` + `NEW`.`amount`;

SET @`sum` = 0;

# 0
SELECT @`sum`;

INSERT INTO `account`
    VALUES (1, 100);

# 100
SELECT @`sum`;

INSERT INTO `account`
    VALUES (2, 500);

# 600
SELECT @`sum`;
```

#### 多语句触发器

### 查看触发器

```SQL
SHOW TRIGGERS;
```

```SQL
SELECT *
    FROM `information_schema`.`TRIGGERS`;
```

```SQL
SELECT *
    FROM `information_schema`.`TRIGGERS`
    WHERE `TRIGGER_NAME` = 'calcSum';
```

### 删除触发器

```SQL
DROP TRIGGER `calcSum`;
```

## 用户

### 创建用户

```SQL
CREATE USER `用户名` IDENTIFIED BY '密码';
```

#### 删除用户

```SQL
DROP USER `jack`;
```

## 权限

### 授予权限

```
GRANT SELECT, UPDATE, ... ON `数据库名`.`数据表名` TO `用户名`;
```