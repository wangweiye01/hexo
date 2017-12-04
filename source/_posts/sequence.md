---
title: mysql自定义sequence
date: 2017-12-04 15:02:51
tags:
---
# 什么是sequence

序列，在Oracle数据库中，什么是序列呢？它的用途是什么？序列(SEQUENCE)其实是序列号生成器，可以为表中的行自动生成序列号，产生一组等间隔的数值(类型为数字)。其主要的用途是生成表的主键值，可以在插入语句中引用，也可以通过查询检查当前值，或使序列增至下一个值。

# mysql没有内置sequence，需要自己实现

## 创建sequence表

```
    CREATE TABLE sequence (
            name VARCHAR(50) NOT NULL,
            current_value INT NOT NULL,
            increment INT NOT NULL DEFAULT 1,
            PRIMARY KEY (name)
            )
```

## 获取当前序列值

```
CREATE FUNCTION currval (seq_name VARCHAR(50))
    RETURNS INTEGER
    LANGUAGE SQL
    DETERMINISTIC
    CONTAINS SQL
    SQL SECURITY DEFINER
    COMMENT ''
    BEGIN
    DECLARE value INTEGER;
    SET value = 0;
    SELECT current_value INTO value
    FROM sequence
    WHERE name = seq_name;
    RETURN value;
    END
```

## 获取下一个序列

```
CREATE FUNCTION nextval (seq_name VARCHAR(50))
    RETURNS INTEGER
    LANGUAGE SQL
    DETERMINISTIC
    CONTAINS SQL
    SQL SECURITY DEFINER
    COMMENT ''
    BEGIN
    UPDATE sequence
    SET current_value = current_value + increment
    WHERE name = seq_name;
    RETURN currval(seq_name);
    END
```

## 重置序列值

```
CREATE FUNCTION setval (seq_name VARCHAR(50), value INTEGER)
    RETURNS INTEGER
    LANGUAGE SQL
    DETERMINISTIC
    CONTAINS SQL
    SQL SECURITY DEFINER
    COMMENT ''
    BEGIN
    UPDATE sequence
    SET current_value = value
    WHERE name = seq_name;
    RETURN currval(seq_name);
    END
```

## 应用

```
    // 在序列表中新建一条序列（参数依次为：序列名称、序列开始值、序列递增步长）
    INSERT INTO sequence VALUES ('TestSeq', 0, 1);
    // 设置序列开始值（参数依次为：序列名称、序列开始值）
    SELECT SETVAL('TestSeq', 10);
    SELECT CURRVAL('TestSeq');
    // 获得下一个序列值
    SELECT NEXTVAL('TestSeq');
```

