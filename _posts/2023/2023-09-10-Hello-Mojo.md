---
title: Mojo
date: 2023-09-10 14:31:57 +0800
categories: [文章]
tags: [Mojo] 
---


我们很高兴通过这款交互式笔记本向您介绍 Mojo！
We're excited to introduce you to Mojo with this interactive notebook!
 
Mojo 被设计为 Python 的超集（Mojo is designed as a superset of Python），因此您在 Python 中可能知道的许多语言功能和概念都可以直接转换为 Mojo。例如，Mojo 中的“Hello World”程序看起来与 Python 完全相同：
Mojo is designed as a superset of Python, so a lot of language features and concepts you might know in Python translate directly to Mojo. For example, a “Hello World” program in Mojo looks exactly like Python:
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1693554153686-b23b9f44-b5d2-4341-bb11-8348c520133d.png#averageHue=%23f9f8f6&clientId=u28296ed2-d475-4&from=paste&height=67&id=u6a7cbcd9&originHeight=67&originWidth=265&originalType=binary&ratio=1&rotation=0&showTitle=false&size=2690&status=done&style=none&taskId=u99330513-91ef-41ea-b45f-875cccbc6f1&title=&width=265)
您还可以导入现有的 Python 包并像使用 Python 编程一样使用它们，但我们稍后会介绍。
You can also import existing Python packages and use them as if you’re programming in Python, but we’ll get to that later.

然而，重要的是要知道 Mojo 本身就是一种全新的语言，而不仅仅是带有额外糖分的 Python 的新实现。随着您对 Mojo 的了解越来越多，您会发现它与 Rust 和 C++ 等语言有更多共同点，除了它使用 Python 语法并完全支持导入的 Python 包。
However, it’s important to know that Mojo is an entirely new language on its own, not just a new implementation of Python with extra sugar. As you learn more about Mojo, you’ll see that it has more in common with languages like Rust and C++, except it uses Python syntax and fully supports imported Python packages.

那么让我们开始吧！本笔记本介绍了 Mojo 语言的基础知识，只需要一点编程经验。
So let's get started! This notebook introduces the basics of the Mojo language, and requires only a little programming experience.
 
如果您想了解有关该语言的更多详细信息，请查看[ Mojo 编程手册](https://docs.modular.com/mojo/programming-manual.html)。
If you want much more detail about the language, check out the [Mojo programming manual](https://docs.modular.com/mojo/programming-manual.html).

:::tips
**Mojo 是一项正在进行的工作：**请通过我们的[ Mojo 社区渠道](https://docs.modular.com/mojo/community.html)向我们发送错误报告、建议和问题。并查看[ Mojo 变更日志](https://docs.modular.com/mojo/changelog.html)中的新增内容。
**Mojo is a work in progress:** Please send us bug reports, suggestions, and questions through our [Mojo community channels](https://docs.modular.com/mojo/community.html). And see what’s new in the [Mojo changelog](https://docs.modular.com/mojo/changelog.html).
:::
:::tips
注意：Mojo Playground 仅为测试 Mojo 语言而设计。云环境并不总是稳定，性能也存在差异，因此不适合进行性能基准测试。然而，我们相信它仍然可以证明 Mojo 提供的性能提升幅度，如 `Matmul.ipynb` 笔记本中所示。有关 Mojo Playground 中计算能力的更多信息，请参阅[ Mojo 常见问题解答](https://docs.modular.com/mojo/faq.html#mojo-playground)。
:::
### Language basics 语言基础知识
首先（First and foremost），Mojo 是一种编译语言，它的许多性能和内存安全功能都源自这一事实。 Mojo 代码可以提前 (AOT) 或即时 (JIT) 编译。 Mojo 还支持[ REPL 环境](https://en.wikipedia.org/wiki/Read–eval–print_loop)，例如在此 Jupyter Notebook 中运行代码的环境（命令行 REPL 即将推出）。
First and foremost, Mojo is a compiled language and a lot of its performance and memory-safety features are derived from that fact. Mojo code can be ahead-of-time (AOT) or just-in-time (JIT) compiled. Mojo also supports [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) environments such as the one that runs the code in this Jupyter notebook (and command-line REPL is coming soon).

与其他编译语言一样，Mojo 需要 `main()` 函数作为程序的入口点。例如：
Like other compiled languages, Mojo requires a `main()` function as the entry point to a program. For example:
```python
fn main():
    var x: Int = 1
    x += 1
    print(x)
```
如果您了解 Python，您可能会期望 `def main()` 而不是 `fn main()`。两者实际上都可以在 Mojo 中使用，但使用 `fn` 的行为有点不同，我们将在下面讨论。
If you know Python, you might have expected `def main()` instead of `fn main()`. Both actually work in Mojo, but using `fn` behaves a bit differently, as we'll discuss below.

 
当然，在 REPL 环境中不需要 `main()` 函数，如上所示，当我们在没有 `main()` 函数的情况下打印“hello world”时。但是当你想编写自己的`.mojo`程序时，`main()`函数是必需的。
Of course, the `main()` function isn't required in a REPL environment, as shown above when we printed "hello world" without a `main()` function. But the `main()` function is required when you want to write your own `.mojo` programs.
:::tips
注意：使用 `.mojo` 文件进行本地开发即将推出 - 目前，使用 Mojo Playground 是运行 Mojo 代码的唯一方法。
**Note:** Local development with `.mojo` files is coming soon — currently, using the Mojo Playground is the only way you can run Mojo code.
:::
现在让我们讨论一下 `main()` 函数中显示的代码。
Now let's discuss the code shown in this `main()` function.

#### Syntax and semantics 语法和语义
这很简单：Mojo 使用 Python 的所有语法和语义。 （如果您不熟悉 Python 语法，网上有大量优质资源可以教您。）
This is simple: Mojo uses all of Python's syntax and semantics. （If you're not familiar with Python syntax, there are a ton of great resources online that can teach you.)

例如，与 Python 一样，Mojo 使用换行符和缩进来定义代码块（不是大括号），并且 Mojo 支持所有 Python 的控制流语法，例如 `if` 条件和 `for` 循环。
For example, like Python, Mojo uses line breaks and indentation to define code blocks (not curly braces), and Mojo supports all of Python's control-flow syntax such as `if` conditions and `for` loops.

然而，Mojo 仍在进行中，因此 Python 中的一些功能尚未在 Mojo 中实现（请参阅 Mojo 路线图）。所有缺失的 Python 功能都会及时到来，但 Mojo 已经包含了许多 Python 之外的特性和功能。
However, Mojo is still a work in progress, so there are some things from Python that aren't implemented in Mojo yet (see the [Mojo roadmap](https://docs.modular.com/mojo/roadmap.html)). All the missing Python features will arrive in time, but Mojo already includes many features and capabilities beyond what's available in Python.

因此，以下部分将重点介绍 Mojo 特有的一些语言功能（与 Python 相比）。
As such, the following sections will focus on some of the language features that are unique to Mojo (compared to Python).

#### Function 功能
 
Mojo 函数可以使用 `fn` （如上所示）或 `def` （如 Python 中所示）来声明。 `**fn**`** 声明强制执行强类型和内存安全行为，而 **`**def**`** 提供 Python 风格的动态行为。**
Mojo functions can be declared with either `fn` (shown above) or `def` (as in Python). The `fn` declaration enforces strongly-typed and memory-safe behaviors, while `def` provides Python-style dynamic behaviors.

 
`fn` 和 `def` 函数都有其价值，学习它们很重要。然而，出于本介绍的目的，我们将仅关注 `fn` 函数。有关两者的更多详细信息，请参阅[编程手册](https://docs.modular.com/mojo/programming-manual.html)。
Both `fn` and `def` functions have their value, and it's important that you learn them both. However, for the purposes of this introduction, we're going to focus on `fn` functions only. For much more detail about both, see the [programming manual](https://docs.modular.com/mojo/programming-manual.html).

 
在以下部分中，您将了解 `fn` 函数如何在代码中强制执行强类型和内存安全行为。
In the following sections, you’ll learn how fn functions enforce strongly-typed and memory-safe behaviors in your code.

#### Variables 变量
您可以声明变量，例如上面 `main()` 函数中的 `x`，**使用 **`**var**`** 创建可变值**或**使用 **`**let**`** 创建不可变值。**
You can declare variables, such as `x` in the above `main()` function, with `var` to create a mutable value or with `let` to create an immutable value.

继续将上面的 `main()` 函数中的 `var` 改为 `let` 并运行它。你会得到这样的编译器错误：
Go ahead and change var to `let` in the `main()` function above and run it. You'll get a compiler error like this:
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1693900548906-fa96904b-6e58-4447-a8ca-6da256937825.png#averageHue=%23fbdbda&clientId=ucb524e13-a5bf-4&from=paste&height=97&id=u8e645b72&originHeight=97&originWidth=665&originalType=binary&ratio=1&rotation=0&showTitle=false&size=6781&status=done&style=none&taskId=u4685ce34-4600-4b30-b932-2ac7fdf1944&title=&width=665)
:::danger
error: Expression [1]:7:5: expression must be mutable for in-place operator destination
x += 1
^  
expression failed to parse (no further compiler diagnostics)
:::
这是因为 `let` 使其不可变，因此您无法增加该值。
That’s because let makes it immutable, so you can’t increment the value.

如果完全删除 var，则会收到错误，因为 fn 函数需要显式变量声明（与 def 函数不同）。
And if you delete var completely, you’ll get an error because fn functions require explicit variable declarations (unlike def functions).

 
最后，请注意 `x` 变量具有显式的 `Int` 类型规范。 `fn` 中的变量不需要声明类型，但有时是需要的。如果省略它，Mojo 会推断类型，如下所示：
Finally, notice that the `x` variable has an explicit `Int` type specification. Declaring the type is not required for variables in `fn`, but it is desirable sometimes. If you omit it, Mojo infers the type, as shown here:
```python
fn do_math():
    let x: Int = 1
    let y = 2
    print(x + y)
do_math()
```
> 3

#### Argument mutability and ownership 参数可变性和所有权
现在让我们探讨如何与函数共享参数值。
Now let’s explore how argument values are shared with a function.

请注意，上面的 `add()` 不会修改 `x` 或 `y`，它只读取值。事实上，正如所写，函数无法修改它们，因为默认情况下 `fn` 参数是不可变的引用。
Notice that, above, `add()` doesn’t modify `x` or `y,` it only reads the values. In fact, as written, the function _cannot_ modify them because `fn` arguments are **immutable references**, by default.

就参数约定而言，这称为“借用”，尽管它是 `fn` 函数的默认设置，但您可以使用像这样的借用声明使其显式化（其行为与上面的 `add()` 完全相同）：
In terms of argument conventions, this is called “borrowing,” and although it's the default for `fn` functions, you can make it explicit with the borrowed declaration like this (this behaves exactly the same as the `add()` above):
```python
fn add(borrowed x: Int, borrowed y: Int) -> Int:
    return x + y;

print(add(1,1))
```
```python
fn add_borrowed(borrowed x: Int, borrowed y: Int) -> Int:
    return x + y

var a = 1
var b = 2
c = add_borrowed(a, b)
print(a)
print(b)
print(c)
```
 ![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1693901940310-e46d1d4a-8564-4b99-8d2e-4ca6b7359be6.png#averageHue=%23fbdbda&clientId=ucb524e13-a5bf-4&from=paste&height=97&id=u39e52e1f&originHeight=97&originWidth=664&originalType=binary&ratio=1&rotation=0&showTitle=false&size=6773&status=done&style=none&taskId=u1e1e3673-09ab-4870-8d90-d7cdc195d18&title=&width=664)
如果您希望参数可变，则需要声明参数约定为 `inout`。这意味着对函数内部参数所做的更改在函数外部是可见的。
If you want the arguments to be mutable, you need to declare the argument convention is `inout`. This means that changes made to the arguments _in_side the function are visible _out_side the function.

例如，该函数可以修改原始变量：
For example, this function is able to modify the original variables:
```python
fn add_inout(inout x: Int, inout y: Int) -> Int:
    x += 1
    y += 1
    return x + y

var a = 1
var b = 2
c = add_inout(a, b)
print(a)
print(b)
print(c)
```
> 2
> 3
> 5

另一种选择是将参数声明为 `owned`，这为函数提供了该值的完全所有权（它是可变的并保证唯一）。这样，函数可以修改值，而不用担心影响函数外部的变量。例如：
Another option is to declare the argument as `owned`, which provides the function full ownership of the value (it's mutable and guaranteed unique). This way, the function can modify the value and not worry about affecting variables outside the function. For example:
```python
fn set_fire(owned text: String) -> String:
    text += "🔥"
    return text

fn mojo():
    let a: String = "mojo"
    let b = set_fire(a)
    print(a)
    print(b)
    
mojo()
```
> mojo
> mojo🔥

在本例中，Mojo 复制 `a` 并将其作为文本参数传递。原来的变量 `a` 依然可用。
In this case, Mojo makes a copy of `a` and passes it as the text argument. The original `a` string is still alive and well.

但是，如果您想赋予函数该值的所有权并且不想进行复制（对于某些类型来说这可能是一个昂贵的操作），那么您可以在将 `a` 传递给函数时添加 `^`“转移”运算符。转移运算符有效地破坏了局部变量名——以后任何调用它的尝试都会导致编译器错误。
However, if you want to give the function ownership of the value and **do not** want to make a copy (which can be an expensive operation for some types), then you can add the `^` “transfer” operator when you pass `a` to the function. The transfer operator effectively destroys the local variable name—any attempt to call upon it later causes a compiler error.
 
通过将 `set_fire()` 的调用更改为如下所示来尝试上面的操作：
```python
fn set_fire(owned text: String) -> String:
    text += "🔥"
    return text

fn mojo():
    let a: String = "mojo"
    let b = set_fire(a^)
    print(a)
    print(b)
    
mojo()
```
  
现在你会得到一个错误，因为**转移运算符 **`**^**`实际上破坏了 `a` 变量，因此当下面的 `print()` 函数尝试使用 `a` 时，该变量不再被初始化。
You'll now get an error because the **transfer operator **`**^**`effectively destroys the a variable, so when the following print() function tries to use a, that variable isn’t initialized anymore.
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1693902836308-46bda026-82ea-4f37-aadd-f7e1143878c1.png#averageHue=%23fcdad8&clientId=ucb524e13-a5bf-4&from=paste&height=153&id=uf607192e&originHeight=153&originWidth=542&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9432&status=done&style=none&taskId=ufdd17498-17c0-4c33-9b84-6b01df36551&title=&width=542)

如果删除 `print(a)`，那么它就可以正常工作。
If you delete `print(a)`, then it works fine.
:::info
**💥  注意：**
目前，当函数返回值时，Mojo 总是创建一个副本。
:::
#### Structures
 
您可以在 `struct` 中为类型（或“对象”）构建高级抽象（higt-level abstractions）。 Mojo 中的 `struct` 类似于 Python 中的类：它们都支持方法、字段、运算符重载、元编程装饰器等。但是，Mojo 结构是完全静态的——它们在编译时绑定，因此不允许动态调度或对结构的任何运行时更改。 （Mojo 将来还将支持课程。）
You can build high-level abstractions for types (or “objects”) in a `struct`. A `struct` in Mojo is similar to a `class` in Python: they both support methods, fields, operator overloading, decorators for metaprogramming, etc. However, Mojo structs are completely static—they are bound at compile-time, so they do not allow dynamic dispatch or any runtime changes to the structure. (Mojo will also support classes in the future.)

例如，这是一个基本结构：
For example, here’s a basic struct:
```python
struct MyPair:
    var first: Int
    var second: Int
    
    # This "initializer" behaves like a constructor in other languages
    fn __init__(inout self, first: Int, second: Int):
        self.first = first
        self.second = second
    
    fn dump(inout self):
        print(self.first)
        print(self.second)
```
使用方法如下：
And here’s how you can use it:
```python
def pair_test() -> Bool:
    let p = MyPair(1, 2)
    # Uncomment to see an error:
    # return p < 4 # gives a compile time error
    return True
```
 
如果您熟悉 Python，那么 `__init__()` 方法和 `self` 参数应该对您很熟悉。如果您不熟悉 Python，请注意，当我们调用 `dump()` 时，我们实际上并没有传递 `self` 参数的值。 `self` 的值是随结构 struct 的当前实例自动提供的（类似于其他语言中使用的 `this`）。
If you're familiar with Python, then the `__init__()` method and the `self` argument should be familiar to you. If you're _not_ familiar with Python, then notice that, when we call `dump()`, we don't actually pass a value for the self argument. The value for `self` is automatically provided with the current instance of the struct (similar to the `this` name used in some other languages).
 
有关结构体 structs 和其他特殊方法（例如 __init__()（也称为“dunder”方法））的更多详细信息，请参阅[编程手册](https://docs.modular.com/mojo/programming-manual.html)。
For much more detail about structs and other special methods like `__init__()` (also known as “dunder” methods), see the [programming manual](https://docs.modular.com/mojo/programming-manual.html).

#### Python integration Python 集成
尽管 Mojo 仍在开发中，并且还不是 Python 的完整超集，但我们已经构建了一种按原样导入 Python 模块的机制，因此您可以立即利用现有的 Python 代码。在底层，该机制使用 CPython 解释器来运行 Python 代码，因此它可以与当今的所有 Python 模块无缝协作。
Although Mojo is still a work in progress and is not a full superset of Python yet, we’ve built a mechanism to import Python modules as-is, so you can leverage existing Python code right away. Under the hood, this mechanism uses the CPython interpreter to run Python code, and thus it works seamlessly with all Python modules today.
 
例如，以下是导入和使用 NumPy 的方法（您必须安装 Python `numpy`，但在本例中，它已安装在 Mojo Playground 中）：
For example, here's how you can import and use NumPy (you must have Python `numpy` installed, but in this case, it’s already installed in the Mojo Playground):
```python
from python import Python

let np = Python.import_module("numpy")

ar = np.arange(15).reshape(3, 5)
print(ar)
print(ar.shape)

# 输出
# [[ 0  1  2  3  4]
# [ 5  6  7  8  9]
# [10 11 12 13 14]]
# (3, 5)
```
:::info
**💥  注意：**
Mojo 还不是 Python 功能完整的超集，因此 Python 中的某些语言模式或功能目前不起作用。因此，您不能总是复制粘贴 Python 代码并在 Mojo 中运行它。[请报告您在 GitHub 上发现的任何问题](https://github.com/modularml/mojo/issues)
:::

### Next steps 下一步
我们希望本笔记本涵盖了足够的基础知识来帮助您入门。它故意简短，所以如果您想了解更多详细信息，请查看[ Mojo 编程手册](https://docs.modular.com/mojo/programming-manual.html)。[另请参阅其他 Mojo 笔记本](https://docs.modular.com/mojo/notebooks/)，了解更有趣和更复杂的代码示例。
We hope this notebook covered enough of the basics to get you started. It’s intentionally brief, so if you want more details, check out the [Mojo programming manual](https://docs.modular.com/mojo/programming-manual.html). Also see the other [Mojo notebooks](https://docs.modular.com/mojo/notebooks/) for more interesting and complex code examples.

要查看所有可用的 Mojo API，请查看[ Mojo 标准库参考](https://docs.modular.com/mojo/lib.html)。
And to see all the available Mojo APIs, check out the [Mojo standard library reference](https://docs.modular.com/mojo/lib.html).
