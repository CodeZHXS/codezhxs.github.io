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
    public static void main(String[] args) {
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

### 数据类型

`Java` 没有方法可以直接查看类型或变量的大小。

#### 整型

| 类型  | 大小  |
| :---: | :---: |
| byte  | 1字节 |
| short | 2字节 |
|  int  | 4字节 |
| long  | 8字节 |

- 没有无符号整型。通常使用`Byte.toUnsignedInt(b)` 将一个有符号的 `byte` 类型 `b` 转换成无符号的 `int` ，以及 `Integer.toUnsignedLong(i);` 来将一个有符号的 `int` 类型 `i` 转换成无符号的 `long` 。
- 类型的大小与机器无关（`java` 是跨平台的）。

#### char类型

- `char` 类型占用 `2` 字节。
- 可以用 `\u` 加上 `4` 个十六进制表示一个 `char`。
- 有些 `Unicode` 可以用一个 `char` 值描述（2字节），有些需要两个 `char` 值（4字节）。
- 

#### Unicode和char类型

- `码点(codepoint)` 就是一个编码表中某个字符对应的代码值。`java` 使用 `Unicode` 编码机制。前缀 `U+` 来表示一个 `Unicode` 字符，例如 `U+0041` 就是 `'A'` 的码点。

- `java` 使用 `UTF-16` 来表示字符，这种编码方式使用 `2` 个字节来表示大多数常用字符，对于其他不太常用的字符使用使用 `4` 个字节表示。

- 每 `2` 个字节称为一个 `代码单元`(code unit) ，在 `java` 中， `char` 类型描述了 `UTF-16` 编码中的一个代码单元。对于大多数字符，`char` 可以直接表示，但某些字符需要两个 `char` 才能表示。因此，不建议在程序中使用 `char` 类型。

#### boolean类型

- 在`java` 中零值不能解析为 `false` ，例如整数的 `0`，对象的 `null` 等。

### 字符串

`java` 中的字符串是 `String` 类型，是 `Unicode` 字符序列。 

- `.substring(l, r)` 方法返回一个 $[l, r)$ 的子串。
- `String` 不支持原地修改。字符串变量的复制实质是共享同一段字符串。
- 不要使用 `==` 检测两个字符串相等，这只能检测两个字符串变量是否共享同一段。如果要检测两个字符串的内容是否相等，要用 `.equals()` 方法。`String` 中还有一个 `.compareTo()` 方法类似于 `C语言` 中的 `strcmp()` 函数。
- 新建一个字符串是 `null` ，`null` 并不是空串的意思。检测空串可以用 `.length() == 0` 或者 `.equals("")` 。对 
- `.length()` 返回代码单元（`char`）的数量，`.codePointCount(l, r)` 可以返回 $[l, r)$ 的码点（`Unicode`） 的数量。对于可能存在 `4` 字节的码点的字符串，遍历时需要注意。

- `StringBuild` 和 `StringBuffer` 是支持原地修改的字符串。其中，`StringBuild` 速度更快，但不是线程安全的。`StringBuffer` 是线程安全的。

### 日期与时间

对于新的代码，应该使用 `java.time` 包的方法。旧的代码可能会使用 `Date` 类。

### 数组

- 将一个数组变量赋给另一个数组变量，实质是引用相同的数组对象。也就是 `浅拷贝` 。深拷贝要用 `Arrays.copyOf()` 方法。
- `java` 的多位数组支持“不规则”，并不要求每一行都有相同的列数。
