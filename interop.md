---
layout: page
title: 互操作指导
site_nav_category: interop
is_site_nav_category: true
site_nav_category_order: 300
---

这是一组关于使用 Java 和 Kotlin 语言编写公共 API 的规则，目的是让代码在其他语言使用时也会感到习惯。

_<a href="changelog.html">上次更新于： {{ site.changes.last.date | date: "%Y-%m-%d" }}</a>_


# Java（被 Kotlin 调用时）

## 不使用硬性关键字

不要使用 Kotlin 的[硬性关键字](https://kotlinlang.org/docs/reference/keyword-reference.html#hard-keywords)作为方法或字段的名称，因此它们会让 Kotlin 在调用时需要使用反引号来避免与其冲突。[软关键字](https://kotlinlang.org/docs/reference/keyword-reference.html#soft-keywords)，[修饰符关键字](https://kotlinlang.org/docs/reference/keyword-reference.html#modifier-keywords)和[特殊标识符](https://kotlinlang.org/docs/reference/keyword-reference.html#special-identifiers)则允许使用。

例如，Mockito 的 `when` 函数在 Kotlin 使用时就需要反引号：

```kotlin
val callable = Mockito.mock(Callable::class.java)
Mockito.`when`(callable.call()).thenReturn(/* … */)
```


## 为空性注解

公共 API 中的每个非原始类型的参数、返回值和字段的类型都应该有一个可为空性的注解。没有这种注解的类型会被当作不确定是否可为空的[“平台”类型](https://kotlinlang.org/docs/reference/java-interop.html#null-safety-and-platform-types)。

JSR 305包注解可用于设置合理的默认值，但目前不建议使用。它们需要一个选择加载的标志位才能被编译器认可，并且还与 Java 9 的模块系统冲突。


## Lambda 参数放在最后

符合 [SAM 转换](https://kotlinlang.org/docs/reference/java-interop.html#sam-conversions)的参数类型应该放到最后。

例如，RxJava 2 的 `Flowable.create()` 方法签名定义为：

```java
public static <T> Flowable<T> create(
    FlowableOnSubscribe<T> source,
    BackpressureStrategy mode) { /* … */ }
```

因为 `FlowableOnSubscribe` 适合进行 SAM 转换，所以 Kotlin 对此方法的函数调用如下所示：

```kotlin
Flowable.create({ /* … */ }, BackpressureStrategy.LATEST)
```

但是，如果方法签名中的参数调换过来，则函数调用可以使用 trailing-lambda 语法：

```kotlin
Flowable.create(BackpressureStrategy.LATEST) { /* … */ }
```


## 属性前缀

对于要在 Kotlin 中表示为属性的方法，必须使用严格的 “bean” 风格的前缀。

访问器方法需要以 “get” 为前缀，如果是 `boolean` 类型的返回方法，使用 “is” 前缀。

```java
public final class User {
  public String getName() { /* … */ }
  public boolean isActive() { /* … */ }
}
```
```kotlin
val name = user.name // 会调用 user.getName()
val active = user.active // 会调用 user.isActive()
```

对应的改值方法则需要 “set” 前缀。

```java
public final class User {
  public String getName() { /* … */ }
  public void setName(String name) { /* … */ }
}
```
```kotlin
user.name = "Bob" // 会调用 user.setName(String)
```

如果您希望将方法公开为属性，请不要使用非标准前缀，如 “has”、“set” 或非 “get” 前缀的访问器方法。具有非标准前缀的方法是否可以接受作为函数调用，则取决于方法的行为。


## 操作符重载

注意在 Kotlin 中允许使用的特殊调用点语法（即[运算符重载](https://kotlinlang.org/docs/reference/operator-overloading.html)）的方法名称。确保方法名称与缩短的语法一起使用是有意义的。

```java
public final class IntBox {
  private final int value;
  public IntBox(int value) {
    this.value = value;
  }
  public IntBox plus(IntBox other) {
    return new IntBox(value + other.value);
  }
}
```
```kotlin
val one = IntBox(1)
val two = IntBox(2)
val three = one + two // Invokes one.plus(two)
```



# Kotlin（被 Java 调用）

## 文件名

当一个文件包含顶级函数或属性时，*始终*用 `@file:JvmName("Foo)"` 注解以提供一个好的名字。

默认情况下，文件 “Foo.kt” 中的顶级成员最终会在一个名为 “FooKt” 的类中，这个类很没有吸引力并且泄漏了语言作为实现的细节。

考虑添加 `@file:JvmMultifileClass` 注解将多个文件中的顶级成员合并为一个类。


## Lambda 参数

要在 Java 中使用的[函数类型](https://kotlinlang.org/docs/reference/lambdas.html#function-types)应避免使用 `Unit` 返回类型。那样做的话就需要指定一条明确的 `return Unit.INSTANCE;` 语句，而这并不符合我们的语言习惯。

```kotlin
fun sayHi(callback: (String) -> Unit) = /* … */
```
```kotlin
// Kotlin caller:
greeter.sayHi { Log.d("Greeting", "Hello, $it!") }
```
```java
// Java caller:
greeter.sayHi(name -> {
    Log.d("Greeting", "Hello, " + name + "!");
    return Unit.INSTANCE;
});
```

这一语法也不允许提供语义命名的类型，以便能在其他类型上实现。

在 Kotlin 中为 lambda 类型定义一个命名的单抽象方法（SAM）接口，可以纠正 Java 的问题，但是也阻止了在 Kotlin 中使用 lambda 语法。

```kotlin
interface GreeterCallback {
    fun greetName(name: String): Unit
}

fun sayHi(callback: GreeterCallback) = /* … */
```
```kotlin
// Kotlin 调用者
greeter.sayHi(object : GreeterCallback {
    override fun greetName(name: String) {
        Log.d("Greeting", "Hello, $name!")
    }
})
```
```java
// Java 调用者
greeter.sayHi(name -> Log.d("Greeting", "Hello, " + name + "!"));
```

在 Java 中定义一个命名的 SAM 接口，允许 Kotlin 使用较低级的  lambda 语法版本，其中必须明确指定接口类型。

```java
// 在 Java 中定义
interface GreeterCallback {
    void greetName(String name);
}
```
```kotlin
fun sayHi(greeter: GreeterCallback) = /* … */
```
```kotlin
// Kotlin 调用者：
greeter.sayHi(GreeterCallback { Log.d("Greeting", "Hello, $it!") })
```
```java
// Java 调用者：
greeter.sayHi(name -> Log.d("Greeter", "Hello, " + name + "!"));
```

目前还没有办法为在 Java 和 Kotlin 使用的 lambda 定义一种参数类型，使得它符合这两种语言的使用习惯。目前的建议是更推荐使用函数类型，尽管当返回类型为 `Unit` 时，在 Java 上体验不佳。

_注意：此建议将来可能会有变化。见 [KT-7770](https://youtrack.jetbrains.com/issue/KT-7770) 和 [KT-21018](https://youtrack.jetbrains.com/issue/KT-21018)。_


## 避免使用 `Nothing` 泛型

泛型参数为 `Nothing` 的类型会作为 Java 的原始类型公开。原始类型很少在 Java 中使用，因此应该避免。


## 记录异常

会抛出检查型异常的函数应该用 `@Throws` 来记录。KDoc 应该记录那些运行异常。

要注意函数委托的 API ，因为它们可能会抛出检查型异常，而 Kotlin 会默许传播这些异常。


## 保护性拷贝

当从公共 API 返回共享或无主的只读集合时，将它们包装在不可修改的容器中或进行保护性拷贝。尽管 Kotlin 会强制执行它们的只读属性，但 Java 方面却没有这样的强制性。如果没有包装器或保护性拷贝，则通过返回长期存在的集合引用将违反不可变性。


## 伴生函数

在 “companion object” 中的公共函数必须用使用 `@JvmStatic` 注解才能暴露为静态方法。

如果没有这个注解，这些函数仅可用作静态 `Companion` 字段上的实例方法。

_不正确：没有注解_

```kotlin
class KotlinClass {
    companion object {
        fun doWork() {
            /* … */
        }
    }
}
```
```java
public final class JavaClass {
    public static void main(String... args) {
        KotlinClass.Companion.doWork();
    }
}
```

_正确：`@JvmStatic` 注解_


```kotlin
class KotlinClass {
    companion object {
        @JvmStatic fun doWork() {
            /* … */
        }
    }
}
```
```java
public final class JavaClass {
    public static void main(String... args) {
        KotlinClass.doWork();
    }
}
```


## 伴生常量

在 `companion object` 中的公共、非 `const` 的属性 [_实际上为常量 _](待补充) 必须用 `@JvmField` 注解才能暴露为静态字段。

如果没有这个注解，这些属性只能作为静态 `Companion` 字段中奇怪命名的 'getters' 实例。而只使用 `@JvmStatic` 而不是 `@JvmField` 的话，会将奇怪命名的 'getters' 移到类的静态方法中，但仍然是不正确的。

_不正确：没有注解_

```kotlin
class KotlinClass {
    companion object {
        const val INTEGER_ONE = 1
        val BIG_INTEGER_ONE = BigInteger.ONE
    }
}
```
```java
public final class JavaClass {
    public static void main(String... args) {
        System.out.println(KotlinClass.INTEGER_ONE);
        System.out.println(KotlinClass.Companion.getBIG_INTEGER_ONE());
    }
}
```

_不正确：`@JvmStatic` 注解_

```kotlin
class KotlinClass {
    companion object {
        const val INTEGER_ONE = 1
        @JvmStatic val BIG_INTEGER_ONE = BigInteger.ONE
    }
}
```
```java
public final class JavaClass {
    public static void main(String... args) {
        System.out.println(KotlinClass.INTEGER_ONE);
        System.out.println(KotlinClass.getBIG_INTEGER_ONE());
    }
}
```

_正确：`@JvmField` 注解_

```kotlin
class KotlinClass {
    companion object {
        const val INTEGER_ONE = 1
        @JvmField val BIG_INTEGER_ONE = BigInteger.ONE
    }
}
```
```java
public final class JavaClass {
    public static void main(String... args) {
        System.out.println(KotlinClass.INTEGER_ONE);
        System.out.println(KotlinClass.BIG_INTEGER_ONE);
    }
}
```

## 命名习惯

Kotlin 有着与 Java 不同的调用约定，这会改变你命名函数的方式。可以使用 `@JvmName` 来设计名称，使得它们对于两种语言的约定都符合习惯，或者匹配它们各自的标准库命名。

这种情况最常出现在扩展功能和扩展属性上，因为接收器类型的位置不同。

```kotlin
sealed class Optional<T : Any>
data class Some<T : Any>(val value: T): Optional<T>()
object None : Optional<Nothing>()

@JvmName("ofNullable")
fun <T> T?.asOptional() = if (this == null) None else Some(this)
```
```kotlin
// 从 KOTLIN 中调用：
fun main(vararg args: String) {
    val nullableString: String? = "foo"
    val optionalString = nullableString.asOptional()
}
```
```java
// 从 JAVA 中调用：
public static void main(String... args) {
    String nullableString = "Foo";
    Optional<String> optionalString =
          Optionals.ofNullable(nullableString);
}
```

## 默认参数的函数重载

具有默认值的参数的函数必须使用 `@JvmOverloads`。如果没有这个注解，则无法使用任何默认值来调用该函数。

当使用 `@JvmOverloads` 时，检查所生成的方法以确保它们都有意义。如果没有的话，请使用以下的一种或两种方法进行重构，直到满意为止：

 1. 更改参数顺序，以让有默认值的参数在结尾。
 2. 将默认值移动到手动重载的函数中。

_不正确：没有 `@JvmOverloads`_

```kotlin
class Greeting {
    fun sayHello(prefix: String = "Mr.", name: String) {
        println("Hello, $prefix $name")
    }
}
```
```java
public class JavaClass {
    public static void main(String... args) {
        Greeting greeting = new Greeting();
        greeting.sayHello("Mr.", "Bob");
    }
}
```

_正确：`@JvmOverloads` 注解。_

```kotlin
class Greeting {
    @JvmOverloads
    fun sayHello(prefix: String = "Mr.", name: String) {
        println("Hello, $prefix $name")
    }
}
```
```java
public class JavaClass {
    public static void main(String... args) {
        Greeting greeting = new Greeting();
        greeting.sayHello("Bob");
    }
}
```



# Lint 检查

## 要求

* **Android Studio 版本：** 3.2 Canary 10 或更高
* **Android Gradle 插件版本：** 3.2.0-alpha10 或更高

## 支持的检查

现在有 Android Lint 检查可以帮助你检测和标记上述一些互操作性的问题。目前仅检测 Java 中的问题（被 Kotlin 调用）。具体来说，所支持的检查有：

* 未知的可为空性
* 属性访问
* 非硬性的 Kotlin 关键字
* Lambda 参数放在最后

## Android Studio

要启用这些检查，请转到 **File > Preferences> Editor > Inspections**，并检查你想要在 Kotlin 互操作性下启用的规则：

<img src="{{ site.baseurl }}/assets/kotlin_interop_checks_settings.png"/>

检查了要启用的规则后，将在运行代码检查（**Analyze > Inspect Code...**）时运行新的检查

## 命令行构建

要从命令行构建中启用这些检查，请在 `build.gradle` 文件中添加以下行：

```groovy
android {

    ...

    lintOptions {
        check 'Interoperability'
    }
}
```

有关 lintOptions 内支持的完整配置，请参阅 [Android Gradle DSL 参考](https://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.LintOptions.html)。

然后，在命令行中运行`./gradlew lint`。
