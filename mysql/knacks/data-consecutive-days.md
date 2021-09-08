#### 1.**需求描述：**

获取连续几天的数据，如果当天没有数据的日期也要展示。

#### 2.**实现方式：**

大致的思路是通过使用MySQL的自定义变量、函数，先拼接出需要查询时间段的日期，然后用这些日期进行left join，无日期的天数给出默认值。

具体实现方式不同的MySQL版本实现方式不同，如在5.6.16版本中使用用户自定义变量，就会报错：

```MySQL
set @a = 1
select @a;

返回结果：
[SQL] select @a;
[Err] 1815 - [20002, 2021090709421419216800006603151853809] : line 1:1: not support variant : @a
```

这时就不能使用自定义变量实现。

而在5.7.28版本中使用用户自定义变量，就不会报错：

```MySQL
set @a = 1;
select @a;

返回结果：
@a
1
```

#### 3.**准备数据：**

```MySQL
CREATE TABLE `user` (
  `id` int(32) NOT NULL AUTO_INCREMENT,
  `name` varchar(32) DEFAULT NULL COMMENT '名称',
  `age` int(8) DEFAULT NULL COMMENT '年龄',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8mb4;

INSERT INTO `user`(`id`, `name`, `age`, `create_time`) VALUES (1, 'zhangsan', 10, '2021-09-05 09:48:10');
INSERT INTO `user`(`id`, `name`, `age`, `create_time`) VALUES (2, 'lisi', 11, '2021-09-06 09:48:16');
INSERT INTO `user`(`id`, `name`, `age`, `create_time`) VALUES (3, 'lisi', 12, '2021-09-08 09:48:20');
```

#### 4.**实现及说明**

查询9月5号-9月10号之间的用户信息，针对**能使用用户自定义变量的版本**实现：

    select DAY, IFNULL(`name`, "") name, IFNULL(`age`, 0) age, IFNULL(`time`, '') time from 
    (SELECT ADDDATE('2021-09-05', INTERVAL @i:=@i+1 DAY) AS DAY
    FROM (
        SELECT a.a
        FROM (SELECT 0 AS a UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) AS a
            CROSS JOIN (SELECT 0 AS a UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) AS b
            CROSS JOIN (SELECT 0 AS a UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) AS c
        ) a
        JOIN (SELECT @i := -1) r1
        WHERE
            @i < DATEDIFF( '2021-09-10', '2021-09-05')
        ) a left join (
            SELECT
                    name, age,
                    date_format(create_time,"%Y-%m-%d") as time
            FROM
                    `user`
            where
                    date_format(create_time,"%Y-%m-%d") >= '2021-09-05'
                            and    date_format(create_time,"%Y-%m-%d") <= '2021-09-10'
        ) b on a.DAY = b.time

说明：

```
// 获得2021-09-05至2021-09-10之间的天数，显示格式为%Y-%m-%d（如：2021-09-05）
// 分析： @i为用户变量，:= 为赋值符号，
// 分析： INTERVAL @i:=@i+1 DAY 意思是天数每次增加1
// 分析： ADDDATE(p1, p2), 为日期p1增加p2个值，p2需用INTERVAL设置，同DATE_ADD('2021-09-05', INTERVAL 1 DAY)
// 分析： a CROSS JOIN b, 连接完的数量为a*b
// 分析： 下面各个CROSS JOIN的sql片段是为了准备1000个0-9的值
// 分析： JOIN (SELECT @i := -1) r1 ，是为了每次查询重新为用户变量赋值
// 分析： DATEDIFF( '2021-09-10', '2021-09-05')，获得两个日期之间的天数，为了控制ADDDATE函数添加的次数
SELECT ADDDATE('2021-09-05', INTERVAL @i:=@i+1 DAY) AS DAY
FROM (
    SELECT a.a
    FROM (SELECT 0 AS a UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) AS a
        CROSS JOIN (SELECT 0 AS a UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) AS b
        CROSS JOIN (SELECT 0 AS a UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) AS c
    ) a
    JOIN (SELECT @i := -1) r1
    WHERE
        @i < DATEDIFF( '2021-09-10', '2021-09-05')
```

查询9月5号-9月10号之间的用户信息，针对**不能使用用户自定义变量的版本**实现：

    select dt, IFNULL(`name`, "") name, IFNULL(`age`, 0) age, IFNULL(`time`, '') time from 
    (SELECT date_sub('2021-09-11', interval 1 day) as  dt
    union all
    SELECT date_sub('2021-09-11', interval 2 day) as  dt
    union all
    SELECT date_sub('2021-09-11', interval 3 day) as  dt
    union all
    SELECT date_sub('2021-09-11', interval 4 day) as  dt
    union all
    SELECT date_sub('2021-09-11', interval 5 day) as  dt
    union all
    SELECT date_sub('2021-09-11', interval 6 day) as  dt
    order by dt desc
        ) a left join (
            SELECT
                    name, age,
                    date_format(create_time,"%Y-%m-%d") as time
            FROM
                    `user`
            where
                    date_format(create_time,"%Y-%m-%d") >= '2021-09-05'
                            and    date_format(create_time,"%Y-%m-%d") <= '2021-09-10'
        ) b on a.dt = b.time

说明：

```
// DATE_SUB() 函数从日期减去指定的时间间隔
SELECT date_sub('2021-09-11', interval 1 day) as  dt
```

#### 5.**扩展：**

通过自定义变量获取连续的月份：

```
// 获得201601至201802之间的月份，显示格式为%Y-%m（如：2016-01）
// 分析： @i为用户变量，：= 为赋值符号，
// 分析： INTERVAL @i:=@i+1 MONTH 意思是月份每次增加1
// 分析： ADDDATE(p1, p2), 为日期p1增加p2个值，p2需用INTERVAL设置，同DATE_ADD('1998-01-02', INTERVAL 3 DAY)
// 分析： a CROSS JOIN b, 连接完的数量为a*b
// 分析： 下面各个CROSS JOIN的sql片段是为了准备1000个0-9的值
// 分析： JOIN (SELECT @i := -1) r1 ，是为了每次查询重新为用户变量赋值
// 分析： PERIOD_DIFF('201802', '201601')，获得两个日期之间的月数，为了控制ADDDATE函数添加的次数

SELECT DATE_FORMAT(ADDDATE('2016-01-06', INTERVAL @i:=@i+1 MONTH),'%Y-%m') AS MONTH
FROM (
    SELECT a.a
    FROM (SELECT 0 AS a UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) AS a
            CROSS JOIN (SELECT 0 AS a UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) AS b
            CROSS JOIN (SELECT 0 AS a UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9) AS c
) a
JOIN (SELECT @i := -1) r1
WHERE
        @i < PERIOD_DIFF('201802', '201601');
```



