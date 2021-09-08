#### **distinct的使用：**

①.只能放在查询语句的最前面表示去重

②.select distinct name, age from a 实际上是根据”name + age“ 来去重（即去除表中name和age这两个字段都相同的记录）

#### **简单验证一下：**

1.创建表a

```MySQL
CREATE TABLE `a` (
  `id` int(32) NOT NULL AUTO_INCREMENT,
  `name` varchar(32) DEFAULT NULL COMMENT '名称',
  `age` int(8) DEFAULT NULL COMMENT '年龄',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8mb4;
```

2.添加数据

```MySQL
INSERT INTO `agent`.`a`(`id`, `name`, `age`) VALUES (1, 'zhangsan', 10);
INSERT INTO `agent`.`a`(`id`, `name`, `age`) VALUES (2, 'lisi', 11);
INSERT INTO `agent`.`a`(`id`, `name`, `age`) VALUES (3, 'lisi', 12);
```

3.执行语句，得出结论

验证①

```MySQL
SELECT `name`, distinct age FROM `A`;

返回：
SELECT `name`, distinct age FROM `A`
1064 - You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'distinct age FROM `A`' at line 1
时间: 0.007s
```

验证：②

```MySQL
SELECT distinct `name`, age FROM `A`;

返回：
name | age
zhangsan 10
lisi 11
lisi 12
```



