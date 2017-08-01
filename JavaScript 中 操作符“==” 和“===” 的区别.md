**标题：JavaScript 中 操作符“==” 和“===” 的区别**

记录一些很坑的区别：

**1.**

```c#
'' == '0'           // false
0 == ''             // true
0 == '0'            // true

false == 'false'    // false
false == '0'        // true

false == undefined  // false
false == null       // false
null == undefined   // true

' \t\r\n ' == 0     // true
```

**2.**

```c#
var a = [1,2,3];
var b = [1,2,3];

var c = { x: 1, y: 2 };
var d = { x: 1, y: 2 };

var e = "text";
var f = "te" + "xt";

a == b            // false
a === b           // false

c == d            // false
c === d           // false

e == f            // true
e === f           // true
```

**3.**

```c#
"abc" == new String("abc")    // true
"abc" === new String("abc")   // false
```

**4.**

>![](https://ws2.sinaimg.cn/large/006tNc79gy1ffz3bdg1ncj31340q4whx.jpg)



**5.**

```c#
11.9.6 The Strict Equality Comparison Algorithm
The comparison x === y, where x and y are values, produces true or false. Such a comparison is performed as follows:

  1. If Type(x) is different from Type(y), return false.
  2. If Type(x) is Undefined, return true.
  3. If Type(x) is Null, return true.
  4. If Type(x) is not Number, go to step 11.
  5. If x is NaN, return false.
  6. If y is NaN, return false.
  7. If x is the same number value as y, return true.
  8. If x is +0 and y is −0, return true.
  9. If x is −0 and y is +0, return true.
  10. Return false.
  11. If Type(x) is String, then return true if x and y are exactly the same sequence of characters (same length and same characters in corresponding positions); otherwise, return false.
  12. If Type(x) is Boolean, return true if x and y are both true or both false; otherwise, return false.
  13. Return true if x and y refer to the same object or if they refer to objects joined to each other (see 13.1.2). Otherwise, return false.
```

**6.两张神图**

![](https://i.stack.imgur.com/62vxI.png)







![](https://i.stack.imgur.com/35MpY.png)

|          值          |   转换为字符串    |  数字  |  布尔值  |          对象           |
| :-----------------: | :---------: | :--: | :---: | :-------------------: |
|      undefined      | "undefined" | NaN  | false |   throws TypeError    |
|        null         |   "null"    |  0   | false |   throws TypeError    |
|        true         |   "true"    |  1   |       |   new Boolean(true)   |
|        false        |   "false"   |  0   |       |  new Boolean(false)   |
|      ""(空字符串)       |             |  0   | false |    new String("")     |
|    "1.2"(非空，数字)     |             | 1.2  | true  |   new String("1.2")   |
|    "one"(非空，非数字)    |             | NaN  | true  |   new String("one")   |
|          o          |     "0"     |      | false |     new Number(0)     |
|         -0          |     "0"     |      | false |    new Number(-0)     |
|         NaN         |    "NaN"    |      | false |    new Number(NaN)    |
|      Infinity       | "Infinity"  |      | true  | new Number(Infinity)  |
|      -Infinity      | "-Infinity" |      | true  | new Number(-Infinity) |
|      1（无穷大，非零）      |     "1"     |      | true  |     new Number(1)     |
|      {}(任意对象)       |    后续讲解     | 后续讲解 | true  |                       |
|      [] (任意数组)      |     ""      |  0   | true  |                       |
|    [9] (一个数字元素)     |     "9"     |  9   | true  |                       |
|    ['a'] (其他数组)     | 使用join() 方法 | NaN  | true  |                       |
| function(){}( 任意函数) |    后续讲解     | NaN  | true  |                       |







参考链接：https://stackoverflow.com/questions/359494/which-equals-operator-vs-should-be-used-in-javascript-comparisons