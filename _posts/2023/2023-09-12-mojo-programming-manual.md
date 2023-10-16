---
title: Mojo 项目手册
date: 2023-09-06 14:31:57 +0800
categories: [文章]
tags: [Mojo] 
---


## Mojo🔥 programming manual Mojo 🔥 项目手册
Mojo 是一种与 Python 一样易于使用的编程语言，但具有 C++ 和 Rust 的性能。此外，Mojo 还提供了利用整个 Python 库生态系统的能力。
Mojo is a programming language that is as easy to use as Python but with the performance of C++ and Rust. Furthermore, Mojo provides the ability to leverage the entire Python library ecosystem.

Mojo 通过利用具有集成缓存、多线程和云分发技术的下一代编译器技术来实现这一壮举。此外，Mojo 的自动调整和编译时元编程功能允许您编写可移植到最奇特硬件的代码。
Mojo achieves this feat by utilizing next-generation compiler technologies with integrated caching, multithreading, and cloud distribution technologies. Furthermore, Mojo’s autotuning and compile-time metaprogramming features allow you to write code that is portable to even the most exotic hardware.

 
更重要的是，**Mojo 允许您利用整个 Python 生态系统**，以便您可以继续使用您熟悉的工具。 Mojo 旨在通过保留 Python 的动态功能同时添加新的系统编程原语（primitives），随着时间的推移成为 Python 的**超集**。这些新的系统编程原语将允许 Mojo 开发人员构建目前需要 C、C++、Rust、CUDA 和其他加速器系统的高性能库。通过将最好的动态语言和系统语言结合在一起，我们希望提供一个统一的编程模型，该模型可以跨抽象级别工作，对新手程序员友好，并且可以扩展到从加速器到应用程序编程和脚本编写的许多用例。
More importantly, **Mojo allows you to leverage the entire Python ecosystem** so you can continue to use tools you are familiar with. Mojo is designed to become a **superset** of Python over time by preserving Python’s dynamic features while adding new primitives for [systems programming](https://en.wikipedia.org/wiki/Systems_programming). These new system programming primitives will allow Mojo developers to build high-performance libraries that currently require C, C++, Rust, CUDA, and other accelerator systems. By bringing together the best of dynamic languages and systems languages, we hope to provide a **unified** programming model that works across levels of abstraction, is friendly for novice programmers, and scales across many use cases from accelerators through to application programming and scripting.

本文档是 Mojo 编程语言的介绍，而不是完整的语言指南。它假定您了解 Python 和系统编程概念。目前，Mojo 仍在开发中，其文档面向具有系统编程经验的开发人员。随着该语言的发展和变得更加广泛，我们希望它对每个人（包括初学者程序员）都友好且易于使用。只是今天不存在了。
This document is an introduction to the Mojo programming language, not a complete language guide. It assumes knowledge of Python and systems programming concepts. At the moment, Mojo is still a work in progress and the documentation is targeted to developers with systems programming experience. As the language grows and becomes more broadly available, we intend for it to be friendly and accessible to everyone, including beginner programmers. It’s just not there today.
### Using the Mojo compiler  使用 Mojo 编译器
您可以从终端运行 Mojo 程序，就像使用 Python 一样。因此，如果您有一个名为 `hello.mojo`（或 `hello.🔥` - 是的，文件扩展名可以是表情符号！）的文件，只需输入 `mojo hello.mojo`：
You can run a Mojo program from a terminal just like you can with Python. So if you have a file named `hello.mojo` (or `hello.🔥`—yes, the file extension can be an emoji!), just type `mojo hello.mojo`:
```bash
$ cat hello.🔥
def main():
	print("hello world")
	for x in range(9, 0, -3):
		print(x)

$ mojo hello.🔥
hello world
9
6
3
$
```
同样，您可以使用 `.🔥` 或 `.mojo` 后缀。
Again, you can use either the `.🔥` or `.mojo` suffix.
### Basic systems programming extensions  基本系统编程扩展
考虑到我们的兼容性目标以及 Python 在高级应用程序和动态 API 方面的优势，我们不必花太多时间来解释该语言的这些部分是如何工作的。另一方面，Python 对系统编程的支持主要委托给 C，我们希望提供一个在这个世界上很棒的单一系统。因此，本节详细介绍了每个主要组件和功能，并通过示例描述了如何使用它们。
Given our goal of compatibility and Python’s strength with high-level applications and dynamic APIs, we don’t have to spend much time explaining how those portions of the language work. On the other hand, Python’s support for systems programming is mainly delegated to C, and we want to provide a single system that is great in that world. As such, this section breaks down each major component and feature and describes how to use them with examples.
#### `let` and `var` declarations `let` 和 `var` 声明
在 Mojo 中的 def 内部，您可以为名称分配一个值，它会隐式创建一个函数作用域变量，就像在 Python 中一样。这提供了一种非常动态且简单的代码编写方式，但它是一个挑战，原因有二：

1.  系统程序员通常希望声明一个值是不可变的，以保证类型安全和性能。
2.  如果他们在赋值中输错了变量名，他们可能希望得到一个错误。

Inside a `def` in Mojo, you may assign a value to a name and it implicitly creates a function scope variable just like in Python. This provides a very dynamic and low-ceremony way to write code, but it is a challenge for two reasons:

1. Systems programmers often want to declare that a value is immutable for type-safety and performance.
2. They may want to get an error if they mistype a variable name in an assignment.

 
为了支持这一点，Mojo 提供了作用域运行时值声明：`**let**`** 是不可变的**，`**var**`** 是可变的**。这些值使用词法作用域（use lexical scoping）并支持名称遮蔽：
To support this, Mojo provides scoped runtime value declarations: `**let**`** is immutable, and **`**var**`** is mutable.** These values use lexical scoping and support name shadowing:
```python
def your_function(a, b):
    let c = a
    # Unconment to see an error
    # c = b # error: c is immutable

    if c != b:
        let d = b
        print(d)

your_function(2, 3)

# results
# 3
```

`let` 和 `var` 声明支持类型说明符和模式以及后期初始化：
`let` and `var` declarations support type specifiers as well as patterns, and late initialization:
```python
def your_function():
    let x: Int = 42
    let y: Float64 = 17.0

    let z: Float32
    if x != 0:
        z = 1.0
    else:
        z = foo()
    print(z)

def foo() -> Float32:
    return 3.14

your_function()

# results
# 1.0
```
 
请注意，let 和 var 在 def 函数中是完全可选的（您可以使用隐式声明的值，就像 Python 一样），但它们对于 fn 函数中的所有变量都是必需的。
Note that `let` and `var` are completely optional when in a `def` function (you can instead use implicitly declared values, just like Python), but they’re required for all variables in an `fn` function.

另请注意，在 REPL 环境（例如本笔记本）中使用 Mojo 时，顶级变量（位于函数或结构之外的变量）会被视为 `def` 中的变量，因此它们允许隐式值类型声明（它们不允许需要 `var` 或 `let` 声明，也不需要类型声明）。这与 Python REPL 行为相匹配。
Also beware that when using Mojo in a REPL environment (such as this notebook), top-level variables (variables that live outside a function or struct) are treated like variables in a `def`, so they allow implicit value type declarations (they do not require `var` or `let` declarations, nor type declarations). This matches the Python REPL behavior.
#### `struct`types 结构类型
Mojo 基于 MLIR 和 LLVM，提供了用于多种编程语言的尖端编译器和代码生成系统。这让我们可以更好地控制数据组织、直接访问数据字段以及其他提高性能的方法。现代系统编程语言的一个重要特征是能够在这些复杂的低级操作之上构建高级且安全的抽象，而不会造成任何性能损失。在 Mojo 中，这是由结构类型提供的。
Mojo is based on MLIR and LLVM, which offer a cutting-edge compiler and code generation system used in many programming languages. This lets us have better control over data organization, direct access to data fields, and other ways to improve performance. An important feature of modern systems programming languages is the ability to build high-level and safe abstractions on top of these complex, low-level operations without any performance loss. In Mojo, this is provided by the struct type.

Mojo 中的结构类似于 Python 类：它们都支持方法、字段、运算符重载、元编程装饰器等。它们的区别如下：

-  Python 类是动态的：它们允许动态调度、猴子修补（或“swizzling”）以及在运行时动态绑定实例属性。
-  Mojo 结构是静态的：它们在编译时绑定（不能在运行时添加方法）。结构允许您以灵活性换取性能，同时安全且易于使用。

A struct in Mojo is similar to a Python class: they both support methods, fields, operator overloading, decorators for metaprogramming, etc. Their differences are as follows:

- Python classes are dynamic: they allow for dynamic dispatch, monkey-patching (or "swizzling"), and dynamically binding instance properties at runtime.
- Mojo structs are static: they are bound at compile-time (you cannot add methods at runtime). Structs allow you to trade flexibility for performance while being safe and easy to use.

 
这是结构体的简单定义：
Here's a simple definition of a struct:
```python
struct MyPair:
    var first: Int
    var second: Int

    # We use 'fn' instead of 'def' here
	# we'll explain that soon
    fn __init__(inout self, first: Int, second: Int):
        self.first = first
        self.second = second

    fn __lt__(self, rhs: MyPair) -> Bool:
        return self.first < rhs.first or
              (self.first == rhs.first and
               self.second < rhs.second)
```
从语法上讲，与 Python `类`相比，最大的区别是`结构`中的所有实例属性都必须使用 `var` 或 `let` 声明显式声明。
Syntactically, the biggest difference compared to a Python class is that all instance properties in a struct **must** be explicitly declared with a `var` or `let` declaration.

 
在 Mojo 中，“struct”的结构和内容是预先设置的，并且在程序运行时无法更改。与 Python 不同，在 Python 中您可以动态添加、删除或更改对象的属性，而 Mojo 不允许对结构进行这种操作。这意味着您不能在程序运行过程中使用 `del` 删除方法或更改其值。
In Mojo, the structure and contents of a "struct" are set in advance and can't be changed while the program is running. Unlike in Python, where you can add, remove, or change attributes of an object on the fly, Mojo doesn't allow that for structs. This means you can't use `del` to remove a method or change its value in the middle of running the program.


#### Strong type checking
#### Overloaded functions and methods
#### fn definitions
#### The __copyinit__ and __moveinit__ special methods

### Argument passing control and memory ownership
#### Why argument conventions are important
#### Immutable arguments (borrowed)
#### Mutable arguments (inout)
#### Transfer arguments (owned and ^)
#### Comparing def and fn argument passing

### Python integration
#### Importing Python modules
#### Mojo types in Python

### Parameterization: compile-time metaprogramming
#### Defining parameterized types and functions
#### Overloading on parameters
#### Using parameterized types and functions
#### Parameter expressions are just Mojo code
#### Powerful compile-time programming
#### Mojo types are just parameter expressions
#### alias: named parameter expressions
#### Autotuning / Adaptive compilation
### "Value Lifecycle": Birth, life and death of a value
#### Types that cannot be instantiated
#### Non-movable and non-copyable types
#### Unique "move-only" types
#### Types that support a “stealing move
#### Trivial types
#### `@value` decorator

### Behavior of destructors
#### Field-sensitive lifetime management
#### Field lifetimes in __init__
#### Field lifetimes of owned arguments in __moveinit__ and __del__
#### Defining the __del__ destructor
### Lifetimes
### Type traits 类型特征
这是一个非常类似于 Rust 特征或 Swift 协议或 Haskell 类型类的功能。请注意，这尚未实施。
This is a feature very much like Rust traits or Swift protocols or Haskell type classes. Note, this is not implemented yet.
### Advanced/Obscure Mojo features 高级/晦涩的 Mojo 功能
#### `@register_passable` struct decorator
#### `@always_inline` decorator
#### `@parameter` decorator
#### Magic operators
#### Direct access to MLIR
#### 







发的
### 
