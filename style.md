---
layout: page
title: 风格指导
site_nav_category_order: 200
is_site_nav_category: true
site_nav_category: style
---

本文档是 Google Android 编码标准的 Kotlin 代码标准。当且仅当一个 Kotlin 源文件符合这里的规则时，我们就称其为 Google Android 代码风格的源文件。

与其他的编程风格指南一样，这里所涉及的问题不仅包括格式化的美观问题，也涉及了其他的约定及编码标准。但是，本文档主要关注我们普遍遵循的硬性规定，并避免给出不明确可执行（无论是通过人工还是工具）的建议。

_<a href="changelog.html">Last update: {{ site.changes.last.date | date: "%Y-%m-%d" }}</a>_

# 源文件

所有源文件的编码必须是 UTF-8。

## 命名

如果一个源文件仅包含一个顶级类，则文件名应该是对应的大小写敏感的名称加上`.kt`扩展名。如果源文件包含多个顶级声明，那么选择一个可以描述文件内容的名称，应用 PascalCase 命名法（即大驼峰命名法），并加上`.kt`扩展名。

```kotlin
// Foo.kt
class Foo { }

// Bar.kt
class Bar { }
fun Runnable.toBar(): Bar = // …

// Map.kt
fun <T, O> Set<T>.map(func: (T) -> O): List<O> = // …
fun <T, O> List<T>.map(func: (T) -> O): List<O> = // …
```

## 特殊字符

### 空白字符

除了行终止符（如回车、换行，译者注），**ASCII 水平空格字符（0x20）**是唯一可以在源文件中任何位置出现的空白字符。这意味着：

 1. 字符串和字符中的所有其他空白字符都会被转义。
 2. 制表符**不会**被用于缩进。

### 特殊转义符

对于任何有[特殊转义符](https://kotlinlang.org/docs/reference/basic-types.html#characters)的字符（`\b`，`\n`，`\r``，`\t`，`\'`，`\"`，`\\`和`\$`），应使用以上字符而不是对应的 Unicode（例如，`\u000a`）转义。

### 非 ASCII 字符

对于其余的非 ASCII 字符，可以使用实际的 Unicode 字符（如`∞`）或者是同义的 Unicode 转义（如`\u221e`）。具体作何选择取决于哪种方式使代码**更易于阅读和理解**。任何位置的可打印字符都不鼓励使用 Unicode 转义，并且强烈建议除了字符串常量和注释之外不要使用 Unicode 转义。

| **例子**                        | **评论**                                                           |
|------------------------------------|--------------------------------------------------------------------------|
| `val unitAbbrev = "μs"`            | 最好：不需要注释也很清晰。                            |
| `val unitAbbrev = "\u03bcs" // μs` | 不好：没有理由使用可打印字符的转义。    |
| `val unitAbbrev = "\u03bcs"`       | 不好：读者不知道这是什么字符。                               |
| `return "\ufeff" + content`        | 可以：对不可打印的字符使用转义，并在必要时进行注释|


## 结构

A `.kt` 文件按顺序包含以下内容：

1. 版本和许可头部（可选）
2. 文件级注解
3. 包语句
4. 导入语句
5. 顶级声明

这里每一个部分都要使用且仅使用一行空行分隔开。

### 版权/许可

如果文件中包含版权或许可标题，应把它们放在多行注释的顶部。

```kotlin
/*
 * Copyright 2017 Google, Inc.
 *
 * ...
 */
 ```

不要对它们使用[KDoc 风格](https://kotlinlang.org/docs/reference/kotlin-doc.html)或单行风格的注释。

```kotlin
/**
 * Copyright 2017 Google, Inc.
 *
 * ...
 */
```
```kotlin
// Copyright 2017 Google, Inc.
//
// ...
```

### 文件级注解

带有 file'[use-site target](https://kotlinlang.org/docs/reference/annotations.html#annotation-use-site-targets) 的注解应放在任何头部注释和包声明之间。

### 包语句

包语句不受行长度限制，并且永远不会换行。

### 导入语句

类，函数和属性的导入语句应组合在一个列表中，并按 ASCII 顺序进行排序。

不允许**（任何类型的）通配符导入**。

与包语句类似，导入语句不受行长度限制，且永不换行。

### 顶级声明

一个`.kt`文件可以在顶级声明一个或多个的类型、函数、属性或类型别名。

一个文件的内容应该集中同一个主题上。这样的例子可以是单个公共类型，或者在多个接收器类型上执行相同操作的一组扩展函数。不相关的声明应该分成它们自己的文件，并且应该最小化单个文件中的 public 声明。

不明确限制文件内容的数量和顺序。

源文件通常是从上往下阅读，这意味着这里的顺序通常应该反映出更高的声明会让人们有更深的理解。不同的文件可以选择对内容进行不同的排序。同样地，一个文件可以包含 100 个属性，10 个函数，以及另外的一个类。

重要的是，每个类都使用**_一些_**维护者可以解释的**逻辑顺序**。比如说，新函数不要只是习惯性地添加到类的末尾，因为这样就会产生“按添加日期排列”的顺序，而这并不是按逻辑排序。

### 类成员顺序

类成员的顺序遵循与顶级声明相同的规则。


# 格式化

## 大括号

当`when`分支和没有'else if`/`else`分支的`if`语句体适合写成单行时，则不需要大括号。

```kotlin
if (string.isEmpty()) return

when (value) {
    0 -> return
    // …
}
```

所有的`if`，`for`，`when`分支，`do`和`while`语句都需要大括号，即使代码体是空的或是只包含一个语句。

```kotlin
if (string.isEmpty())
    return  // 错误的！

if (string.isEmpty()) {
    return  // 正确
}
```

### 非空代码块

对于非空代码块和块状结构，大括号遵循 Kernighan 和 Ritchie 风格（“埃及括号”）：

* 左大括号前不需要换行
* 在左大括号之后另起一行
* 在右大括号前换行
* 只有在右大括号结束一个语句、一个函数体、构造方法或_非匿名_类时，才在右大括号之后换行。例如，当后面有`else`语句或逗号时，则右大括号不需要换行。

```kotlin
return Runnable {
    while (condition()) {
        foo()
    }
}

return object : MyClass() {
    override fun foo() {
        if (condition()) {
            try {
                something()
            } catch (e: ProblemException) {
                recover()
            }
        } else if (otherCondition()) {
            somethingElse()
        } else {
            lastThing()
        }
    }
}
```

下面是关于[枚举类](#enum-classes）的一些例外情况。

### 空代码块

一个空代码块或块状结构必须采用 K&R 风格。

```kotlin
try {
    doSomething()
} catch (e: Exception) {} // 错误的！
```
```kotlin
try {
    doSomething()
} catch (e: Exception) {
} // 正确
```

### 表达式

如果`if`/`else`条件用于表达式，那么_只有_当整个表达式适合一行时，才可以省略大括号。

```kotlin
val value = if (string.isEmpty()) 0 else 1  // 正确
```
```kotlin
val value = if (string.isEmpty())  // 错误的！
                0
            else
                1
```
```kotlin
val value = if (string.isEmpty()) { // Okay
    0
} else {
    1
}
```


## 缩进

每次打开一个新的代码块或块状结构时，缩进增加四个空格。当块结束时，回到先前的缩进级别。缩进级别对整个块中的代码和注释都适用。


## 一行一个语句

每一个语句之后都要断行，并且不使用分号。


## 换行

代码的列限制为100个字符。除了下面说明的情况，任何超出此限制的行都必须换行，如下所述。

例外情况：

* 不能遵守列限制的行（例如，KDoc中的长URL）
* `package`和`import`语句
* 注释中的命令行可以被剪切并粘贴到 shell 中

### 换行位置

换行的主要原则是：优先在**更高的语法级别**上换行。包括：

 1. 当要在_非赋值_运算符处换行时，在符号_之前_进行断行。
    * 这也适用于以下“类似运算符”的符号：
      * 点分割符（`.`）
      * 成员引用的两个冒号（`::`）
 2. 当需要在_赋值_运算符处断开时，在符号_之后_进行断行。
 3.方法或构造器的名称要紧跟其左括号（`(`）。
 4. 逗号（`,`）应紧跟之前的内容。
 5. lambda 箭头（`->`）要紧跟它之前的参数列表。

注意：换行的主要上的是让代码更清晰，而_不一定_要让代码符合最小行数。


### 继续缩进

换行时，第一行之后的每一行（即每一个_连续行_）从原始行上缩进至少8个空格。

当有多个连续行时，缩进可以根据需要增加到超过 +8。通常情况下，当且仅当两个连续行以语法并行的元素开头时，它们使用相同的缩进级别。

### 函数

当一个函数签名不适合单行时，每个参数声明为一行。以此格式定义的参数应使用单个缩进（+4）。右括号（`)`）和返回类型一起放在新的一行，并且没有额外的缩进。

```kotlin
fun <T> Iterable<T>.joinToString(
    separator: CharSequence = ", ",
    prefix: CharSequence = "",
    postfix: CharSequence = ""
): String {
    // …
}
```

#### 表达式函数

当函数只包含单个表达式时，它可以表示为[表达式函数](https://kotlinlang.org/docs/reference/functions.html#single-expression-functions)。

```kotlin
override fun toString(): String {
    return "Hey"
}
```
```kotlin
override fun toString(): String = "Hey"
```

表达式函数不应换行成两行。如果表达式函数增长到需要换行，则应使用普通函数体，`return`声明和正常表达式换行规则来代替。

### 属性

当属性的初始化不适合单行时，在等号（`=`）后断开并使用继续缩进。

```kotlin
private val defaultCharset: Charset? =
        EncodingRegistry.getInstance().getDefaultCharsetForPropertiesFiles(file)
```

声明了`get`或`set`函数的属性应当在它们各自的行上使用正常的缩进（+4）。使用与函数相同的规则格式化它们。

```kotlin
var directory: File? = null
    set(value) {
        // …
    }
```

只读属性可以使用适合单行的较短语法。

```kotlin
val defaultExtension: String get() = "kt"
```


## 空白

### 纵向

一个空行应该出现在：

 1. 在类的连续成员_之间_：属性，构造器，函数，嵌套类等。

     * **例外**：两个连续属性之间（它们之间没有其他代码）的空行是可选的。根据需要，可以使用空行对属性进行逻辑分组，将这些属性与其支持属性​​（如果有）关联。

     * **例外**：枚举常量之间的空行如下所示。

 2. 在语句之间，_需要_将代码组织成逻辑子部分。

 3. 在函数中的第一个语句之前，类的第一个成员之前，或者是在类的最后一个成员之后的空行是可选的（既不鼓励也不反对）。

 4. 按照本文档其他部分的要求（如[“结构”](#structure)部分）。

允许多个连续空行，但不鼓励或不需要。

### 横向

除了语言或其他风格规则的要求，以及字面量、注释和 KDoc 之外，单个 ASCII 空格也只应该出现在以下位置：

 1. 将任何一个保留字如`if`、`for`或`catch`和同一行内跟随着它的左括号（`(`）分隔开。

    ```kotlin
    // 错误的！
    for(i in 0..1) {
    }
    ```
    ```kotlin
    // 正确
    for (i in 0..1) {
    }
    ```

 2. 将任何一个保留字（如“else”或“catch”）与同一行位于它之前的右大括号（`}`）分隔开。

    ```kotlin
    // 错误的！
    }else {
    }
    ```
    ```kotlin
    // 正确
    } else {
    }
    ```

 3. 在所有的左大括号（`{`）之前。

    ```kotlin
    // 错误的！
    if (list.isEmpty()){
    }
    ```
    ```kotlin
    // 正确
    if (list.isEmpty()) {
    }
    ```

 4. 在任何二元运算符的两边。

    ```kotlin
    // 错误的！
    val two = 1+1
    ```
    ```kotlin
    // 正确
    val two = 1 + 1
    ```

    这也适用于以下“类似运算符”的符号：

     *  lambda 表达式中的箭头（`->`）。

        ```kotlin
        // 错误的！
        ints.map { value->value.toString() }
        ```
        ```kotlin
        // 正确
        ints.map { value -> value.toString() }
        ```

    但不适用于：

     * 成员引用的两个冒号（`::`）。

        ```kotlin
        // 错误的！
        val toString = Any :: toString
        ```
        ```kotlin
        // 正确
        val toString = Any::toString
        ```

     *  点分割符（`.`）。

        ```kotlin
        // 错误的
        it . toString()
        ```
        ```kotlin
        // 正确
        it.toString()
        ```

    *  范围运算符（`..`）。

        ```kotlin
        // 错误的
        for (i in 1 .. 4) print(i)
        ```
        ```kotlin
        // 正确
        for (i in 1..4) print(i)
        ```

 5. 只有用于类声明中指定基类或接口，或者是[泛型约束](https://kotlinlang.org/docs/reference/generics的.html#gereric-constraints)的`where`子句的冒号前面。

    ```kotlin
    // 错误的！
    class Foo: Runnable
    ```
    ```kotlin
    // 正确
    class Foo : Runnable
    ```
    
    ```kotlin
    // 错误的
    fun <T: Comparable> max(a: T, b: T)
    ```
    ```kotlin
    // 正确
    fun <T : Comparable> max(a: T, b: T)
    ```

    ```kotlin
    // 错误的
    fun <T> max(a: T, b: T) where T: Comparable<T>
    ```
    ```kotlin
    // 正确
    fun <T> max(a: T, b: T) where T : Comparable<T>
    ```

 6. 在逗号（`,`）或冒号（`:`）之后。

    ```kotlin
    // 错误的！
    val oneAndTwo = listOf(1,2)
    ```
    ```kotlin
    // 正确
    val oneAndTwo = listOf(1, 2)
    ```

    ```kotlin
    // 错误的！
    class Foo :Runnable
    ```
    ```kotlin
    // 正确
    class Foo : Runnable
    ```

 7. 在行尾注释的双斜杠（`//`）的两边。这里允许多个空格，但不作要求。

    ```kotlin
    // 错误的！
    var debugging = false//disabled by default
    ```
    ```kotlin
    // 正确
    var debugging = false // disabled by default
    ```

这条规则不会被解释为在行的开头或结尾处要求或禁止额外的空格；它只涉及行内的空格。


## 具体结构

### 枚举类

没有函数并且没有关于其常量的文档的枚举，可以格式化为单行。

```kotlin
enum class Answer { YES, NO, MAYBE }
```

当枚举中的常量放在单独的行上时，它们之间不需要空行，除非它们定义了代码体。

```kotlin
enum class Answer {
    YES,
    NO,

    MAYBE {
        override fun toString() = """¯\_(ツ)_/¯"""
    }
}
```

由于枚举类也是类，因此适用于格式化类的所有其他规则。

### 注解

成员或类型的注解要作为单独一行放在所注解的结构之前。

```kotlin
@Retention(SOURCE)
@Target(FUNCTION, PROPERTY_SETTER, FIELD)
annotation class Global
```

没有参数的注解可以放在同一行上。

```kotlin
@JvmField @Volatile
var disposable: Disposable? = null
```

当只有一个注解并且没有参数时，可以与声明放在同一行。

```kotlin
@Volatile var disposable: Disposable? = null

@Test fun selectAll() {
    // …
}
```

### 隐式返回/属性类型

如果一个表达式函数体或属性初始化是一个标量值，或者可以从函数体明确推断出返回类型，则类型可以省略。

```kotlin
override fun toString(): String = "Hey"
// 成为
override fun toString() = "Hey"
```
```kotlin
private val ICON: Icon = IconLoader.getIcon("/icons/kotlin.png")
// 成为
private val ICON = IconLoader.getIcon("/icons/kotlin.png")
```

编写库时，如果它是公共 API 的一部分，请保留显式类型声明。


# 命名

标识符仅使用 ASCII 字母和数字，并且在下面提到的少数情况下使用下划线。因此，每个有效的标识符名称都由正则表达式“\ w +”匹配。

除了支持属性（参见[“Backing properties”](#backing-properties)）之外，不使用特殊前缀或后缀，如`name_`，`mName`，`s_name`和`kName`。


## 包名

包名都是小写的，并且连续的单词简单地连接在一起（没有下划线）。

```kotlin
// 正确
package com.example.deepspace
// 错误的！
package com.example.deepSpace
// 错误的！
package com.example.deep_space
```


## 类名

类使使用 PascalCase 命名，并且通常是名词或名词短语。例如，`Character`或`ImmutableList`。界面名称也可以是名词或名词短语（如`List`），但有时也可以是形容词或形容词短语（如`Readable`）。

测试类的名称以所测试的类的名称开头，以“Test”结尾。比如`HashTest`或`HashIntegrationTest`。


## 函数名

函数名使用 camelCase（即小驼峰命名法，译者注）命名，且通常是动词或动词短语。比如`sendMessage`或`stop`。

允许下划线出现在测试函数名中，以分隔名称里的逻辑组件。

```kotlin
@Test fun pop_emptyStack() {
    // …
}
```


## 常量名

常量名使用 UPPER_SNAKE_CASE 命名法：全部大写字母，单词用下划线分隔。但是，什么_才是_一个常量？

常量是指没有自定义 `get` 函数的 `val` 属性，其内容完全不可变，并且其函数没有明显的副作用。这包括不可变类型和不可变类型的不可变集合，以及标记为 `const` 的标量和字符串。如果实例的某一个可观察状态可以被改变，则它就不是常量。只是打算永远不改变该对象也是不符合条件的。

```kotlin
const val NUMBER = 5
val NAMES = listOf("Alice", "Bob")
val AGES = mapOf("Alice" to 35, "Bob" to 32)
val COMMA_JOINER = Joiner.on(',') // Joiner 是不可改变的
val EMPTY_ARRAY = arrayOf<SomeMutableType>()
```

这些名称通常是名词或名词短语。

常量值只能在 `object` 中或作为顶级声明定义。否则满足常量要求但在 `class` 中定义的值必须使用非常量名称。

标量值常量必须使用 [`const` 修饰符](http://kotlinlang.org/docs/reference/properties.html#compile-time-constants)。


## 非常量名

非常量名是使用 camelCase 命名法命名，它们适用于实例属性，本地属性和参数名称。

```kotlin
val variable = "var"
val nonConstScalar = "non-const"
val mutableCollection: MutableSet<String> = HashSet()
val mutableElements = listOf(mutableInstance)
val mutableValues = mapOf("Alice" to mutableInstance, "Bob" to mutableInstance2)
val logger = Logger.getLogger(MyClass::class.java.name)
val nonEmptyArray = arrayOf("these", "can", "change")
```

这些名称通常是名词或名词短语。

### 支持属性

当需要[支持属性](https://kotlinlang.org/docs/reference/properties.html#backing-properties)时，其名称应与实际的名称完全匹配，但是有下划线前缀。

```kotlin
private var _table: Map<String, Int>? = null

val table: Map<String, Int>
    get() {
        if (_table == null) {
            _table = HashMap()
        }
        return _table ?: throw AssertionError()
    }
```


## 类型变量名

类型变量使用以下两种风格之一进行命名：

 1. 一个大写字母，后面可以跟着一个数字（如`E`，`T`，`X`，`T2`）
 2.用于类的形式的名字，后面跟着大写字母`T`（例如 `RequestT`，`FooBarT`）


## 驼峰命名法

有些时候可以有多种方式将英语短语转换为驼峰大小写，比如当存在缩略词或者是 “IPv6” 或 “iOS” 等不寻常的结构时。为了提高可预测性，请使用以下方案进行命名。

从名称的散文形式开始：

 1. 将短语转换为纯 ASCII 并删除任何撇号。例如，“Müller's algorithm” 会变成 “Muellers algorithm”。

 2. 将上一步的结果按空格和所有剩下的标点符号（通常为连字符）分割成多个单词。

    * _推荐_：如果其中的单词已经是通常使用的常见的驼峰形式，将其拆分为其组成部分（比如 “AdWords” 变为 “ad words”）。注意，像 “iOS” 这一类的单词_本身_并不是真的以驼峰命名；它违背了_任何_惯例，因此不适用于这个建议。

 3. 现在把所有内容都转为小写（包括首字母缩略词），然后：

    * ...对每一个单词的第一个字符都大写，以产生大驼峰；或者是

    * ...除了第一个单词以外，其他单词的第一个字符都大写，以产生小驼峰

 4. 最后，将所有单词连接成一个标识符。

注意，原始单词的大小写几乎完全被忽略。

| **Prose form**         | **Correct**                             | **Incorrect**       |
|------------------------|-----------------------------------------|---------------------|
| "XML Http Request"     | `XmlHttpRequest`                        | `XMLHTTPRequest`    |
| "new customer ID"      | `newCustomerId`                         | `newCustomerID`     |
| "inner stopwatch"      | `innerStopwatch`                        | `innerStopWatch`    |
| "supports IPv6 on iOS" | `supportsIpv6OnIos`                     | `supportsIPv6OnIOS` |
| "YouTube importer"     | `YouTubeImporter`<br>`YoutubeImporter`* |                     |

（_*表示可接受，但不推荐_）

**注意**：某些英文单词对连接字符的使用是不明确的：比如 “nonempty” 和 “non-empty” 都是对的，因此方法名 `checkNonempty` 和 `checkNonEmpty` 也同样都是对的。


# 文档


## 格式化

如下示例中可以看到 KDoc 块的基本格式：

```kotlin
/**
 * 在这里写多行 KDoc 文本，
 * 正常换行…
 */
fun method(arg: String) {
    // …
}
```

...或者是以下的单行形式：

```kotlin
/** 特别短的 KDoc. */
```

上面基本形式都是可以接受的。当整个 KDoc 块（包括注释标记）可以写在一行上时，可以使用单行形式。请注意，这仅适用于没有如 `@ return` 这样的块标签的情况。

### 段落

一个空白行——即只包含对齐的前导星号（`*`）的行——在段落之间以及块标签组（如果有）之前出现。

### 块标签

任何标准的“块标签”的使用都以 `@constructor`，`@receiver`，`@param`，`@property`，`@return`，`@throws`，`@see` 的顺序出现，并且它们不能有空的描述。当块标记不适合单行时，连续行从 `@` 的位置缩进 8 个空格。


## 摘要片段

每个 KDoc 块都以简短的摘要片段开头。这个片段非常重要：它是文本中唯一出现在某些上下文（如类和方法索引）中的部分。

它是一个片段——一个名词短语或动词短语，而不是一个完整的句子。它不是以“``A `Foo` is a…``”或“`This method returns…`”开头，也不必形成一个完整的祈使句如“`Save the record.`”。然而，这个片段是大写开头并且有标点的，就像它是一个完整的句子一样。


## 用法

至少每种“public”类型，以及这种类型的每个“public”或“protected”成员要有 KDoc，除了下面列出的例外情况。

### 例外：自解释的函数

对于“简单、明显”的函数或属性，KDoc 是可选的。比如像 `getFoot` 函数或者是 `foo` 属性，在某些情况下，除了“Returns the foo”之外真的没有什么值得说的。

引用这个例外并不适合证明可以省略一般读者可能需要知道的相关信息。例如，对于名为 `getCanonicalName` 的函数或名为 `canonicalName` 的属性，如果一般读者可能不知道术语“规范名称”是什么意思的话，那就不要省略它的文档（省略的理由是它只会说`/** Returns the canonical name. */`）！

### 例外：重写

如果一个方法是重写父类的方法，那么可以不需要 KDoc。
