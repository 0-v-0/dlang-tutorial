# 控制流

控制流是编程语言中用于控制程序执行顺序的机制。通过使用条件语句和循环语句，可以根据不同的条件和需求来控制程序的执行流程。

## 条件语句

条件语句用于根据条件的真假来决定程序的执行路径。

### if语句

if语句用于根据条件的真假来执行不同的代码块。语法如下：

```d
if (condition) {
    // 如果条件为真，执行这里的代码
} else {
    // 如果条件为假，执行这里的代码
}
```

示例：

```d
int num = 10;

if (num > 0) {
    writeln("num是正数");
} else {
    writeln("num是负数或零");
}
```

if语句条件中可以声明变量，该变量的作用域为if语句块

示例：

```d
if (int r = n % 10) {
    // ...
}
```


### switch语句

switch语句用于根据不同的值执行不同的代码块。语法如下：

```d
switch (expr) {
    case value1:
        // 如果expr等于value1，执行这里的代码
        break;
    case value2:
        // 如果expr等于value2，执行这里的代码
        break;
    default:
        // 如果expr不等于任何一个case的值，执行这里的代码
        break; // 最后一个break可省略
}
```

示例：

```d
int num = 2;

switch (num) {
    case 1:
        writeln("num等于1");
        break;
    case 2:
        writeln("num等于2");
        break;
    default:
        writeln("num不等于1或2");
}
```

多个语句上相邻的case可以合并：例如
```d
case 1:
case 2:
case 3:
```
可简化为`case 1, 2, 3:`

范围case：例如`case 1: .. case 3:`相当于`case 1, 2, 3:`

#### Fall-Through
在D语言中不允许隐式Fall-Through，一个非空case的所有执行路径上必须存在[改变控制流的语句](#改变控制流的语句)或[死循环](#死循环)

如果需要继续执行，必须使用`goto case;`显式Fall-Through

### final switch语句
final switch语句除了以下几点区别外，其他和switch语句相同
1. 不能有default语句
2. 不能有范围case
3. 如果表达式是枚举类型，则所有枚举成员必须出现在必须出现在case中
4. case表达式的值必须能在编译时计算出来

默认编译参数下，如果表达式不匹配任何case的值，将抛出`SwitchError`（运行时）

## 循环语句

循环语句用于重复执行一段代码，直到满足特定条件为止

### while循环

while循环用于重复执行一段代码，只要条件为真就会一直执行。语法如下：

```d
while (condition) {
    // 循环体
}
```

示例：

```d
int i = 0;

while (i < 5) {
    writeln(i);
    i++;
}
```

### do-while循环

do-while循环用于重复执行一段代码，先执行一次循环体，然后再判断条件是否为真。如果条件为真，则继续执行循环体，否则退出循环。语法如下：

```d
do {
    // 循环体
} while (condition);
```

示例：

```d
int i = 0;

do {
    writeln(i);
    i++;
} while (i < 5);
```

### for循环

for循环用于重复执行一段代码，可以指定循环的起始条件、终止条件和每次迭代后的操作。语法如下：

```d
for (initialization; condition; increment) {
    // 循环体
}
```

示例：

```d
for (int i = 0; i < 5; i++) {
    writeln(i);
}
```

### break和continue语句

在循环中，可以使用break语句来提前退出循环，使用continue语句来跳过本次循环，进入下一次循环。

示例：

```d
for (int i = 0; i < 5; i++) {
    if (i == 3) {
        break; // 提前退出循环
    }
    writeln(i);
}

for (int i = 0; i < 5; i++) {
    if (i == 3) {
        continue; // 跳过当前循环
    }
    writeln(i);
}
```

break和continue后面可以加标签


### foreach循环

foreach循环用于遍历可迭代对象中的元素。语法如下：

```d
foreach (element; iterable) {
    // 循环体
}
```

如果需要反向遍历，可以把foreach替换成改为foreach_reverse

可迭代对象有以下几种：
1. 数组（包括关联数组）
2. 区间，例如`0 .. 10`表示区间`[0, 10)`
3. 包含`empty`、`front`和`popFront`（对于foreach），或`back`和`popBack`（对于foreach_reverse）方法的对象
4. 包含`opApply`方法的对象

示例：

```d
int[] numbers = [1, 2, 3, 4, 5];

foreach (int num; numbers) {
    writeln(num);
}
```

## goto语句

goto语句用于无条件地跳转到程序中的标签位置。在D语言中，一般情况下不推荐使用goto语句（goto case除外），因为可能导致程序的控制流变得复杂和难以理解。

例如：
```d
uint nextPrime(uint n) {
	loop: for (;;) {
		n++;
		for (uint i = 2; i * i <= n; i++) {
			if (n % i == 0)
				continue loop;
		}
		return n;
	}
}
```
这个函数用goto语句可简化为
```d
uint nextPrime(uint n) {
    next: n++;
    for (uint i = 2; i * i <= n; i++) {
        if (n % i == 0)
            goto next;
    }
	return n;
}
```

goto不能跳过目标标签所在作用域内变量的初始化，以下例子会导致编译错误
```d
if (condition)
    goto label;

auto s = S(42);

label:
    s.bar();
```

## 死循环
若一个循环的循环条件在编译时恒为真，且循环体中没有break或return语句，则称这个循环为死循环

下面是一个goto语句形成的死循环：
```d
loop:
    writeln(1);
goto loop;
```

### 改变控制流的语句

以下语句会无条件改变当前程序控制流：
1. goto
2. return
3. break
4. continue
5. 调用noreturn函数
6. throw表达式
7. 一个条件在编译时恒为假的assert语句

goto case属于goto语句的一种
