# 函数的分类
## 数值类函数
**round(x[, d]) - round x to d decimal places**
```
    对x保留d位小数，同时四舍五入。
```
**floor(x) - Find the largest integer not greater than x**
```
    找出一个不大于x的最大整数
```

**ceil(x) - Find the smallest integer not smaller than x**
```
    找出一个不大于x的最小的整数
```
## 数学函数
**abs(x) - returns the absolute value of x**
```
    返回x的绝对值
```
**pow(x1, x2) - raise x1 to the power of x2**
```
    x1的x2次幂
```
## 日期函数
## 条件函数
**if**
```
    IF(expr1,expr2,expr3) - If expr1 is TRUE (expr1 <> 0 and expr1 <> NULL) then IF() returns expr2; otherwise it returns expr3. IF() returns a numeric or string value, depending on the context in which it is used.
    如果expr1为true则返回expr2，否则返回expr3
```
**case when**
```
    相当于switch case
```
## 字符串函数
**instr(str, substr)**
```
    返回字符第一次出现的索引
```
**concat(str1, str2, ... strN)**
```
    select concat("l","j","k")
    return:ljk 
```
**concat_ws(separator, [string | array(string)]+)**
```
    select concat_ws("/",["l","j","k"]);
    return:l/j/k
```
## 统计函数
**index(a, n)**
```
     select index(array(1,2,3,4,5),3);
     return:4
```
**sum/min/count/avg/max**
## 特殊函数
**array**
```
    返回一个数组
```
**collect_set**
```
    返回一个set集合
```
**explode(a)**
```
    行转列
```

## 自定义函数
**UDF**
```
    用户定义函数
    一路输入，一路输出
```
**UDAF**
```
    用户自定义聚合函数
    多路输入，一路输出
```
**开窗函数**