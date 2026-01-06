# js引入方式
![[Pasted image 20260106123045.png]]

# js基础语法
## 书写语法
![[Pasted image 20260106123659.png]]
![[Pasted image 20260106123855.png]]

## 变量
![[Pasted image 20260106124149.png]]
var的特点
- 特点1：作用域比较大，全局变量。即使是在代码块内定义，也可以在代码块之外调用。
- 特点2：可以重复定义

==注意事项==
- ECMAScript6 新增了==let== 关键字来定义变量。它的用法类似于 var，但是所声明的变量，只在let 关键字所在的代码块内有效，且不允许重复声明。
- ECMAScript6 新增了 ==const== 关键字，用来声明一个只读的常量。一旦声明，常量的值就不能改变。

## 数据类型、运算符
JavaScript中分为:原始类型 和引用类型。

==原始类型==
- number:数字(整数、小数、NaN(Not a Number))
- string:字符串，单双引皆可
- boolean:布尔。true，false
- null:对象为空
- undefined:当声明的变量未初始化时，该变量的默认值是 undefined

==类型转换==
- 字符串类型转为数字：
	- 将字符串字面值转为数字。如果字面值不是数字，则转为
	- alert(parseInt("12"));//12
	- alert(parseInt("12A45"));//12 （会一直识别到不是数字的地方）
	- alert(parseInt("A45"));//NaN   (not a number）
- 其他类型转为boolean:
	- Number：0和NaN为false，其他均转为true。
	- String:空字符串为false，其他均转为true。
	- Null和undefined :均转为false。
# js函数
![[Pasted image 20260106125957.png]]
![[Pasted image 20260106130249.png]]

# js对象


# js事件监听

