JavaScript 中的类型转换



**1.在处理财务或科学数据时**

 有如下三个方法，toFixed()， toExponential()，toPrecision()。

toFixed() 根据小数点后的指定位数将数字转换为字符串，它从不使用指数计数法。toExponential() 使用指数计数法将数字转换成指数形式的字符串，其中小数点前只有一位，小数点后的位数则由参数指定。toPrecision() 根据指定的有效数字位数将数字转换成字符串。如果有效数字的位数少于数字整数部分的位数，则转换成指数形式。

**所有三个方法都会适当地进行四舍五入或填充0**

例：

```c#
var n = 123456.789;
n.toFixed(0); //123457
n.toFixed(2); //123456.79
n.toFixed(5); //123456.78900

n.toExponential(1); //1.2e+5
n.toExponential(3); //1.235e+5

n.toPrecision(4); //1.235e+5
n.toPrecision(7); //123456.8
n.toPrecision(10); //123456.7890
```

