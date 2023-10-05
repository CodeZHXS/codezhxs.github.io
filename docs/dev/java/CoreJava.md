## 第2章	Java程序设计环境

### 下载JDK

[Latest Releases | Adoptium](https://adoptium.net/zh-CN/temurin/releases/?os=linux&arch=x64)

### 设置环境变量

```bash
export JAVA_HOME=/usr/local/jdk-11.0.20.1+1 #jdk包的位置
export PATH=$JAVA_HOME/bin:$PATH #添加命令行工具

# 这个变量是用于全局的寻找CLASS的路径 不建议开启
# export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

```bash
❯ java --version
openjdk 11.0.20.1 2023-08-24
OpenJDK Runtime Environment Temurin-11.0.20.1+1 (build 11.0.20.1+1)
OpenJDK 64-Bit Server VM Temurin-11.0.20.1+1 (build 11.0.20.1+1, mixed mode)
```

### 编译运行

通常认为 `java` 是介于 `编译型` 和 `解释型` 之间的编程语言。具体来说，可以使用 `javac` 命令将 `.java` 文件编译成 `.class` 文件， `.class` 是字节码，不是二进制可执行文件，而是要使用 `java` 命令，将 `.class` 交给 `JVM` 解释执行。

新建一个 `Hello.java` 文件：

```java
public class Hello {
    public static void main(String[] args) 
    {
        System.out.println("Hello, World!");
    }
}
```

执行以下指令：

```bash
❯ javac Hello.java
❯ ls
Hello.class  Hello.java
❯ java Hello      
Hello, World!
```

### CLASSPATH

上面的程序虽然可以直接使用 `java` 命令直接运行：

```bash
❯ ls
Hello.java
❯ java Hello.java                   
Hello, World!
```

但是，只有包含main函数、且类依赖关系非常简单的文件才可以这样做。大多数情况下，都是使用 `javac` 命令生成 `.class` 文件，然后用 `java` 命令执行。

在使用 `javac` 命令而不指定 `-d` 参数时，生成的 `.class` 文件会默认生成在当前目录下。比较好的习惯是将项目的源代码和二进制文件分开存放，可以用 `-d` 参数指定 `.class` 文件的存放的路径，例如这个项目路径：

```
.
├── bin
├── lib
└── src
    └── Hello.java
```

在这个项目结构中，`src` 存放所有源代码文件，`bin` 存放所有 `.class` 文件。

使用 `javac -d` 在指定目录中生成 `.class` 文件：

```bash
❯ javac src/Hello.java -d bin/ 
```

 此时文件结构变成

```
.
├── bin
│   └── Hello.class
├── lib
└── src
    └── Hello.java
```

可以 `cd` 到 `bin` 中然后再 `java` 执行（注：可以尝试一下，是不能直接在外面 `java bin/Hello` 的，原因是 `java` 命令严格按照 `1级包名/2级包名/.../类名` 的格式解析，所以这条命令会将`bin/`解析为一个包(`package`)的名字, 而`Hello.java` 中并没有没有指定任何`package` ）：

```bash
❯ cd bin
❯ java Hello
Hello, World!
```

另一种方法是指定 `CLASSPATH`，只需要在执行 `java` 命令时增加 `-cp` 参数：

```ba
❯ java -cp bin Hello  
Hello, World!
```

上述指令指定了 `bin` 作为 `CLASSPATH` ,   这样就会在 `bin` 目录下寻找 `.class` 文件，咋一看这种方式和 `cd` 没有区别，但是， `-cp` 可以指定多个路径作为 `CLASSPATH`，（在Linux中可以用 `:` 将多个路径隔开），并且可以设置全局的 `CLASSPATH` 环境变量（但不推荐这样做）。

### **IDE** 如何简化以上流程？

许多现代的 **IDE** 可以简化上述的流程，例如，为每个项目添加 `CLASSPATH`，并且随着文件的改动 `自动编译`，相当于帮我们动态的执行 `javac -d bin ...` ，也能 `自动管理文件` ，例如清理不再依赖的类等。我们只需要点击 `run` 按钮，就会自动执行 `java -cp bin ...` 的命令。当然，**IDE** 能做的不止这些，常见的 `java` **IDE** 有 `IDEA`、`Eclipse` 等，使用 `VSCode` 并配置一些插件也可以达到 **IDE** 的开发体验。

### JShell

从 `Java 9` 开始，在命令行输入 `jshell` 即可进入交互式编程环境：

```bash
❯ jshell
jshell> "Core Java".length()
$1 ==> 9
jshell> 5 * $1 - 3
$2 ==> 42
jshell> int answer = 5 * $1 - 3
answer ==> 42
jshell> Math.log10(0.001)
$4 ==> -3.0
```

---

## 第3章	Java的基本程序设计结构

`Java` 没有方法可以直接查看类型或变量的大小。

### 整型

| 类型  | 大小  |
| :---: | :---: |
| byte  | 1字节 |
| short | 2字节 |
|  int  | 4字节 |
| long  | 8字节 |

- 没有无符号整型。通常使用`Byte.toUnsignedInt(b)` 将一个有符号的 `byte` 类型 `b` 转换成无符号的 `int` ，以及 `Integer.toUnsignedLong(i);` 来将一个有符号的 `int` 类型 `i` 转换成无符号的 `long` 。
- 类型的大小与机器无关（`java` 是跨平台的）。

### char类型

- `char` 类型占用 `2` 字节。
- 可以用 `\u` 加上 `4` 个十六进制表示一个 `char`。
- 有些 `Unicode` 可以用一个 `char` 值描述（2字节），有些需要两个 `char` 值（4字节）。

### Unicode和char类型

- `码点(codepoint)` 就是一个编码表中某个字符对应的代码值。`java` 使用 `Unicode` 编码机制。前缀 `U+` 来表示一个 `Unicode` 字符，例如 `U+0041` 就是 `'A'` 的码点。

- `java` 使用 `UTF-16` 来表示字符，这种编码方式使用 `2` 个字节来表示大多数常用字符，对于其他不太常用的字符使用使用 `4` 个字节表示。

- 每 `2` 个字节称为一个 `代码单元`(code unit) ，在 `java` 中， `char` 类型描述了 `UTF-16` 编码中的一个代码单元。对于大多数字符，`char` 可以直接表示，但某些字符需要两个 `char` 才能表示。因此，不建议在程序中使用 `char` 类型。

### boolean类型

- 在`java` 中零值不能解析为 `false` ，例如整数的 `0`，对象的 `null` 等。

### 字符串

`java` 中的字符串是 `String` 类型，是 `Unicode` 字符序列。 

- `.substring(l, r)` 方法返回一个 $[l, r)$ 的子串。
- `String` 不支持原地修改。字符串变量的复制实质是共享同一段字符串。
- 不要使用 `==` 检测两个字符串相等，这只能检测两个字符串变量是否共享同一段。如果要检测两个字符串的内容是否相等，要用 `.equals()` 方法。`String` 中还有一个 `.compareTo()` 方法类似于 `C语言` 中的 `strcmp()` 函数。
- 新建一个字符串是 `null` ，`null` 表示没有引用任何对象，并不是空串的意思。检测空串可以用 `.length() == 0` 或者 `.equals("")` 。
- `.length()` 返回代码单元（`char`）的数量，`.codePointCount(l, r)` 可以返回 $[l, r)$ 的码点（`Unicode`） 的数量。对于可能存在 `4` 字节的码点的字符串，遍历时需要注意。

- `StringBuild` 和 `StringBuffer` 是支持原地修改的字符串。其中，`StringBuild` 速度更快，但不是线程安全的。`StringBuffer` 是线程安全的。

### 日期与时间

对于新的代码，应该使用 `java.time` 包来表示时间点。旧的代码可能会使用 `Date` 类来表示时间点。

### 数组

- 将一个数组变量赋给另一个数组变量，实质是引用相同的数组对象。也就是 `浅拷贝` 。 `深拷贝` 要用 `Arrays.copyOf()` 方法。
- `java` 的多维数组支持“不规则”，并不要求每一行都有相同的列数。

---

## 第4章 对象与类

### 对象变量是引用

`java` 中的对象变量类似于 `C++` 中的对象指针，对象变量实质是引用某个对象，所以新建一个对象变量必须要用 `new 构造函数()` 。`java` 中不能得到一个对象本身，所有的对象都是在堆中构造的。

构造函数总是与 `new` 配合使用，不能对一个已经存在的对象调用构造器来打到重新设置实例字段的目的。如果要重设实例字段，还是需要配合 `new` 使用，这会引用一个新的对象。

某些类的对象并不使用构造函数来构造，而是使用 `静态工厂方法` , 例如`LocalDate`类（用来表示某个日期）使用 `LocalDate.now()` 和 `LocalDate.of()`，调用这些方法直接就能获得一个对象的引用（不需要 `new` ）。

### 构造函数

#### 默认字段初始化

- 数值型的默认值为 `0`，布尔型默认值为 `false`，对象引用默认值为 `null`。
- 如果一个类没有构造函数，则默认会提供一个无参数的构造函数将所有实例字段设置为默认值。
- 如果一个类已经写了至少一个构造函数，则不会提供默认的无参数构造函数。
- 在构造函数中没有被赋值的字段会被赋为默认值。

#### 重载

`java` 支持重载语法，包括构造函数和其他函数都可以重载。

#### null 值处理

直接定义一个对象变量（而不调用构造函数）会得到一个 `null` 。对 `null` 使用方法会产生异常。

调用构造函数时，可以对传入的参数检查，假设有这样的类：

```java
class Employee {
    private String name;
    private double salary;
    private LocalDate hireDay;
}
```

其中 `name` 和 `hireDay` 都有可能传入 `null` 值。对于可能传入的 `null` 值，第一种处理方法是赋一个默认对象变量，在 `java 9` 中， `Objects` 类提供了一个 `.requireNonNullElse()` 方法（所有类都扩展自 `Objects` 类）：

```java
public Employee(String n, double s, int year, int month, int day) {
    name = Objects.requireNonNullElse(n, "unknown");
    ...
}
```

使用这个方法，当 `n == null` 时会赋默认值 `name = "unknown"` 。

第二种处理是借助`.requireNonNull()` 方法，可以在 `n == null` 时抛出异常：

```java
public Employee(String n, double s, int year, int month, int day) {
    Objects.requireNonNull(n, "The name cannot be null");
}
```

#### 调用另一个构造函数

使用 `this` 关键字， 可以在一个构造函数中调用另一个构造函数：

```java
public Employee(double s) {
    // 假设之前定义了 Employee（String n, double s）
    this("Employee #" + nextId, s);
    nextId++;
}
```

#### final

类似于 `C++` 中的 `const`，用来表示常量，定义后不可修改。

```java
final int maxn = 1010; // 不能更改的常量
```

实例字段如果设置为 `final` ，则必须在构造函数中设置 `final` 字段的值，并且设置之后不能更改。

### 初始化块

一共有三种初始化数据字段的方式，按照顺序为：

1. 在声明时直接赋值。
2. 在初始化块中赋值。
3. 在构造函数中赋值。

如果构造函数的第一行调用了另一个构造函数，则会先执行第二个构造函数。否则，先执行初始化块。初始化块可以有多个，会按照顺序执行。

### 静态字段与静态方法

`static` 表示 `静态字段`，同一个类的所有对象都共享同一个字段。`静态字段` 通过类名调用。

例如要给每个员工分配一个递增的 `id`，则 `nextID` 变量用于记录下一个分配的 `id` 值。

```java
class Employee {
    private static int nextID = 1;
    private int id;
    public void setId() {
        id = nextID++;
    }
}
```

`static final` 表示 `静态常量`，例如 `Math.PI`。

`静态方法` 是指可以通过类名直接调用的方法，例如 `Math.pow()` 。

`静态工厂方法` 是指可以通过类名直接调用的构造函数，例如 `LocalDate.now()` 和 `LocalDate.of()` 。

`main()` 函数的缩写是 `psvm` ，即 `public static void main(String[] args) `。每个类可以有一个 `main` 函数，用于对类进行单元测试。单元测试只会运行单独的一个 `main` 函数，例如 `java Employee` 就是运行 `Employee.class` 的 `main` 函数。

### `java` 的参数传递是按值引用的

`java` 只有按值传递参数的方式，因此：

- 方法不能修改基本数据类型的参数（即数值型或布尔型）。
- 方法可以改变对象参数的状态。
- 方法不能让一个对象参数引用一个新的对象。

### 包

#### 类的导入

一个类可以使用所属包中的所有类，以及其他包中的公共类。

有两种方法访问其他包中的公共类：

1. 使用完全限定名，例如：

```java
java.time.LocalDate today = java.time.LocalDate.now();
```

2. 使用 `import` 导入整个包或者特定的类，例如：

```java
import java.time.LocalDate; //导入特定的类
```

或者：

```java
import java.time.* //导入整个包
```

注意只能使用使用 `*` 导入一个包，例如不能使用 `import java.*` ，因为 `java.*` 不止一个包。

`java` 中的 `import` 类似于 `C++` 中的 `using namespace` ，因为 `java` 编译器可以查看其他文件的内容，而 `C++` 编译器不行，所以在 `C++` 中需要使用 `#include` 。

#### 静态导入

使用 `import static` 可以导入静态方法和静态字段，例如：

```java
import static java.lang.Math.sqrt;
public class Main {
    public static void main(String[] args) {
        double ans = sqrt(2.0);
        System.out.println(ans);
    }    
}
```

#### 包路径

包和目录要匹配，虽然编译器编译时不检查目录结构，但是虚拟机运行时会按照包的结构去寻找类。所以推荐的做法就是按照包的结构设置文件的结构。

#### JAR文件

可以将 `.class` 文件打包在一个 `.jar` 文件中， `.jar` 文件实质就是 `.zip` 文件，这样可以节省空间、改善性能。

在使用第三方的库文件时，通常都会得到一个或者多个 `.jar` 文件。在以下这种项目结构中，通常将`.jar` 文件放在 `lib` 文件夹中。

```
.
├── bin
├── lib
└── src
```

在运行时，需要用 `:`将 `lib` 文件夹添加到 `CLASSPATH` 中，这样虚拟机

### 类设计技巧

1. 一定要保证数据私有。
2. 一定要对数据进行初始化。
3. 不要在类中使用过多的基本类型（当基本类型过多时，考虑用一个新的类型替换相关的字段）。
4. 不是所有的字段都需要单独的访问器和更改器。
5. 分解有过多指责的类。
6. 类名和方法名要能够体现它们的功能。
7. 优先使用不可变的类（例如 `LocalDate` 类就是不可变的，这样可以保证线程安全）。
