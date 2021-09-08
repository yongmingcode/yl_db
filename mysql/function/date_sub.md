#### 函数用法介绍

DATE\_SUB\(date,INTERVAL expr type\) 函数从日期date减去对应的时间间隔expr

date 参数是合法的日期表达式。 expr 参数是您希望减掉的时间间隔。

type 参数可以是下列值：

| MICROSECOND |
| :--- |
| SECOND |
| MINUTE |
| HOUR |
| DAY |
| WEEK |
| MONTH |
| QUARTER |
| YEAR |
| SECOND\_MICROSECOND |
| MINUTE\_MICROSECOND |
| MINUTE\_SECOND |
| HOUR\_MICROSECOND |
| HOUR\_SECOND |
| HOUR\_MINUTE |
| DAY\_MICROSECOND |
| DAY\_SECOND |
| DAY\_MINUTE |
| DAY\_HOUR |
| YEAR\_MONTH |

#### 实例：

```MySQL
SELECT DATE_SUB('2021-09-05', INTERVAL 1 DAY) as dt

返回结果：
dt
2021-09-04
```



