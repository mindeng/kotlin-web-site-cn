---
type: doc
layout: reference
title: "操作符符重载"
category: "Syntax"
---

# 操作符符重载

Kotlin允许我们实现一些我们自定义类型的运算符实现。这些操作符有固定的表示
（像 `+` 或者 `*`），和固定的[优先级](grammar.html#precedence)。为实现这样的操作符，我们提供了固定名字的[成员函数](functions.html#member-functions)
或[扩展函数](extensions.html)，比如二元操作符的左值和一元操作符的参数类型。
Functions that overload operators need to be marked with the `operator` modifier.

## 转换

这里我们描述了一些常用操作符的重载。

### 一元操作符

| 表达式 | 翻译为 |
|------------|---------------|
| `+a` | `a.unaryPlus()` |
| `-a` | `a.unaryMinus()` |
| `!a` | `a.not()` |

这张表解释了当编译器运行时，比如，表达式 `+a` ，是这样运行的：

* 决定`a`的类型, 假设为`T`。
* 寻找接收者是`T`带有`operator`修饰的无参函数`unaryPlus()`,例如一个成员方法或者扩展方法。
* 如果找不到或者不明确就返回一个错误。
* 如果函数是当前函数或返回类型是`R`则表达式`+a`是`R`类型。

*注意* 这些操作符和其它的一样, 都被优化为[基本类型](basic-types.html)并且不会产生多余的开销。

| 表达式 | 翻译为 |
|------------|---------------|
| `a++` | `a.inc()` + 见下方 |
| `a--` | `a.dec()` + 见下方 |


这些操作符允许修改接收者和返回类型。

> **`inc()/dec()` 不应该改变接收对象**.<br>
> "修改接收者"你应该修改接收者变量而非对象。
{:.note}

编译器是这样解决有*后缀*的操作符的比如`a++`:

* 决定`a`的类型, 假设为`T`。
* 查找接收类型为`T`带有`operator`修饰的无参数函数`inc()。
* 如果返回类型为`R`,那么`R`为`T`子类型.

计算表达式的步骤是：

* 把`a`的值存在`a0`中,
* 把`a.inc()`结果作用于 `a`,
* 把 `a0`作为表达式的结果.

a-- 的运算步骤也是一样的。

对于前缀操作符`++a`和`--a`的解决方式也是一样的, 步骤是:

* 把`a.inc()`作用于`a`,
* 返回新值`a`作为表达式结果。

### 二元操作符

| 表达式 | 翻译为 |
| -----------|-------------- |
| `a + b` | `a.plus(b)` |
| `a - b` | `a.minus(b)` |
| `a * b` | `a.times(b)` |
| `a / b` | `a.div(b)` |
| `a % b` | `a.mod(b)` |
| `a..b ` | `a.rangeTo(b)` |

编译器只是解决了该表中翻译为列的表达式。

| Expression | Translated to |
| -----------|-------------- |
| `a in b` | `b.contains(a)` |
| `a !in b` | `!b.contains(a)` |

in 和 !in 的产生步骤是一样的，但参数顺序是相反的。
{:#in}

| 符号 | 翻译为 |
| -------|-------------- |
| `a[i]`  | `a.get(i)` |
| `a[i, j]`  | `a.get(i, j)` |
| `a[i_1, ...,  i_n]`  | `a.get(i_1, ...,  i_n)` |
| `a[i] = b` | `a.set(i, b)` |
| `a[i, j] = b` | `a.set(i, j, b)` |
| `a[i_1, ...,  i_n] = b` | `a.set(i_1, ..., i_n, b)` |

方括号被转换为 get set 函数。

| 符号 | 翻译为 |
|--------|---------------|
| `a()`  | `a.invoke()` |
| `a(i)`  | `a.invoke(i)` |
| `a(i, j)`  | `a.invoke(i, j)` |
| `a(i_1, ...,  i_n)`  | `a.invoke(i_1, ...,  i_n)` |

括号被转换为带有正确参数的 `invoke` 函数。

| 表达式 | 翻译为 |
|------------|---------------|
| `a += b` | `a.plusAssign(b)` |
| `a -= b` | `a.minusAssign(b)` |
| `a *= b` | `a.timesAssign(b)` |
| `a /= b` | `a.divAssign(b)` |
| `a %= b` | `a.modAssign(b)` |
{:#assignments}

在分配 a+= b时编译器是下面这样实现的:

* 右边函数是否可用。
  * 对应的二元函数是否 (如`plus()`和`plusAssign()`)也可用, 不可用就报告错误.
  * 确定它的返回值是`Unit`类型, 否则报告错误。
  * 生成`a.plusAssign(b)`
*否则试着生成`a = a + b`代码 (这里包含类型检查: `a + b`一定要是`a`的子类型).

*注意*: assignments在Kotlin中不是表达式.
{:#Equals}

| 表达式 | 翻译为 |
|------------|---------------|
| `a == b` | `a?.equals(b) ?: (b === null)` |
| `a != b` | `!(a?.equals(b) ?: (b === null))` |

*注意*: `===` 和 `!==` (实例检查)不能重载, 所以没有转换方式。

The `==` 操作符有点特殊: 它被翻译成一个复杂的表达式，用于筛选`null`值，而且 `null == null` 是 `true`。

| 符号 | 翻译为 |
|--------|---------------|
| `a > b`  | `a.compareTo(b) > 0` |
| `a < b`  | `a.compareTo(b) < 0` |
| `a >= b` | `a.compareTo(b) >= 0` |
| `a <= b` | `a.compareTo(b) <= 0` |

所有的比较都转换为`compareTo`的调用，这个函数需要返回`Int`值

## 命名函数的中缀调用

我们可以通过[中缀函数的调用](functions.html#infix-notation) 来模拟自定义中缀操作符。