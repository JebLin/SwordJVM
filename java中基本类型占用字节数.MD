
###  在Java中一共有8种基本数据类型，其中有4种整型，2种浮点类型，1种用于表示Unicode编码的字符单元的字符类型和1种用于表示真值的boolean类型。（一个字节等于8个bit）

#### 1.整型
```
  类型 存储需求 bit数 取值范围 备注
  int 4字节 4*8
  short 2字节 2*8 －32768～32767
  long 8字节 8*8
  byte 1字节 1*8 －128～127
```
#### 2.浮点型
```
类型 存储需求 bit数 取值范围 备注
float 4字节 4*8 float类型的数值有一个后缀F(例如：3.14F)
double 8字节 8*8 没有后缀F的浮点数值(如3.14)默认为double类型
```
#### 3.char类型
```
类型 存储需求 bit数 取值范围 备注
char 2字节 2*8 一个中文字符占两个byte（一个char），英文字符可以转换为byte，而 （byte）‘中’ 会出错。
```
#### 4.boolean类型
```
类型 存储需求 bit数 取值范围 备注
boolean 1字节 1*8 false、true
```

补充：Java有一个能够表示任意精度的算书包，通常称为“大数值”(big number)。虽然被称为大数值，但它并不是一种Java类型，而是一个Java对象。
如果基本的整数和浮点数精度不能够满足需求，那么可以使用java.math包中的两个很有用的类：BigIntegerBigDecimal(Android SDK中也包含了java.math包以及这两个类)这两个类可以处理包含任意长度数字序列的数值。BigInteger类实现了任意精度的整数运算，BigDecimal实现了任意精度的浮点数运算。具体的用法可以参见Java API。

来源： http://blog.csdn.net/witsmakemen/article/details/8974200
