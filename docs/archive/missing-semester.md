课程讲义：[计算机教育中缺失的一课 · the missing semester of your cs education (missing-semester-cn.github.io)](https://missing-semester-cn.github.io/)

视频链接（中文字幕）：[刘黑黑a的个人空间-刘黑黑a个人主页-哔哩哔哩视频 (bilibili.com)](https://space.bilibili.com/518734451)

## 第 1 讲 - 课程概览与 shell

本讲介绍了 `Shell` 的基础知识以及常见的命令。

一些不太常见的用法：

-   `tee` 命令用于输出一段文字同时将输出的内容写入到某个文件中。可以用来打日志。
-   用 双引号`"` 包裹起来的字符串内部会转译，而用单引号 `'` 包裹起来的字符串所见即所得。

### 课后练习

>   1.  本课程需要使用类Unix shell，例如 Bash 或 ZSH。如果您在 Linux 或者 MacOS 上面完成本课程的练习，则不需要做任何特殊的操作。如果您使用的是 Windows，则您不应该使用 cmd 或是 Powershell；您可以使用[Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/)或者是 Linux 虚拟机。使用`echo $SHELL`命令可以查看您的 shell 是否满足要求。如果打印结果为`/bin/bash`或`/usr/bin/zsh`则是可以的。

```bash
echo $SHELL
```

>   2.   在 `/tmp` 下新建一个名为 `missing` 的文件夹。

```bash
mkdir /tmp/missing
ls /tmp
```

>   3.   用 `man` 查看程序 `touch` 的使用手册。

```bash
man touch
```

>   4.   用 `touch` 在 `missing` 文件夹中新建一个叫 `semester` 的文件。

```bash
touch /tmp/missing/semester
ls /tmp/missing
```

>   5.   将以下内容一行一行地写入 `semester` 文件：
>
>        ```
>        #!/bin/sh
>        curl --head --silent https://missing.csail.mit.edu
>        ```
>
>        第一行可能有点棘手， `#` 在Bash中表示注释，而 `!` 即使被双引号（`"`）包裹也具有特殊的含义。 单引号（`'`）则不一样，此处利用这一点解决输入问题。更多信息请参考 [Bash quoting 手册](https://www.gnu.org/software/bash/manual/html_node/Quoting.html)

```bash
cd /tmp/missing
echo '#!/bin/sh' > semester
echo 'curl --head --silent https://missing.csail.mit.edu' >>  semester
cat semester
```

>   6.   尝试执行这个文件。例如，将该脚本的路径（`./semester`）输入到您的shell中并回车。如果程序无法执行，请使用 `ls` 命令来获取信息并理解其不能执行的原因。

```bash
./semester
```

会提示没有权限。

>   7.   查看 `chmod` 的手册(例如，使用 `man chmod` 命令)

```bash
man chmod
```

>   8.   使用 `chmod` 命令改变权限，使 `./semester` 能够成功执行，不要使用 `sh semester` 来执行该程序。您的 shell 是如何知晓这个文件需要使用 `sh` 来解析呢？更多信息请参考：[shebang](https://en.wikipedia.org/wiki/Shebang_(Unix))

```bash
chmod +x semester
./semester
```

>   9.   使用 `|` 和 `>` ，将 `semester` 文件输出的最后更改日期信息，写入主目录下的 `last-modified.txt` 的文件中

```bash
./semester | grep 'last-modified' > ~/last-modified.txt
cat ~/last-modified.txt
```

---

## 第 2 讲 - Shell 工具和脚本

本讲介绍了 `Shell` 脚本以及一些工具。

### Shell 脚本

#### 特殊变量

`Shell` 脚本使用一些特殊的符号来表示参数、错误代码和相关变量。下面是一些比较常用的用法：

|     符号     |            含义            |
| :----------: | :------------------------: |
|     `$0`     |           脚本名           |
| `$1` 到 `$9` |         脚本的参数         |
|     `$@`     |         所有的参数         |
|     `$#`     |         参数的个数         |
|     `$?`     |     上一条命令的错误码     |
|     `$$`     |       当前脚本的 pid       |
|     `!!`     | 完整的上一条命令，包括参数 |
|     `$_`     |  上一条命令的最后一个参数  |

#### 一个函数的例子

```bash
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```

#### 输出流与错误流

-   程序的输出流用 `>` 重定向，错误流用 `2>` 重定向。

-   脚本之间通过错误码来交流执行的结果，错误码为 `0` 表示没有错误，非 `0` 表示有错误。

-   错误码可以搭配 `&&` 和 `||` 使用。
-   可以将输出流和错误流重定向到 `/dev/null`。

#### 一个 Shell 脚本的例子

```bash
#!/bin/bash

echo "Starting program at $(date)" # date会被替换成日期和时间

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do # $@展开成列表
    grep foobar "$file" > /dev/null 2> /dev/null
    # 如果模式没有找到，则grep退出状态为 1
    # 我们将标准输出流和标准错误流重定向到Null，因为我们并不关心这些信息
    # 在bash中进行比较时，尽量使用双方括号 [[ ]] 而不是单方括号 [ ]
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```

#### 通配符与花括号

-   `*` 表示匹配任意个任意字符，`?` 表示匹配一个任意字符。

-   使用花括号 `{}` 可以对相同的子串进行自动展开。这在批量操作时很方便。

    展开的时候用空格分开，例如：

    ```bash
    convert image.{png,jpg}
    # 展开为
    convert image.png image.jpg
    ```

-   在 `zsh` 中，对花括号语句按 `tab` 就会自动展开，但是即使不展开来，包含花括号的语句也是能直接运行的。`bash` 中不能按 `tab` 自动展开。

-   多个 会按笛卡尔积展开。例如：

    ```bash
    mkdir -p touch project{1,2}/src/test/test{1,2,3}
    # 展开为
    mkdir -p touch project1/src/test/test1 project1/src/test/test2 project1/src/test/test3 project2/src/test/test1 project2/src/test/test2 project2/src/test/test3
    ```

-   在花括号 `{}` 中可以用 `..` 来表示数字或字母的递增。例如：

    ```bash
    mkdir foo bar
    touch {foo,bar}/{a..j}.cpp
    # 会在 foo 和 bar 下都创建 a.cpp b.cpp ... j.cpp
    ```

#### 在 Shell 中运行 Python 脚本

通常需要用 `python name.py` 来运行一个 `Python` 脚本。但是如果在脚本的第一行加上解释器的路径，并给 `Python` 脚本加上可执行权限，就能像运行 `Shell` 脚本一样运行 `Python` 脚本。

例如：

```python
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```

### Shell 工具

#### tldr

使用 `tldr` 加上任意命令，用来查找这个命令的经典用法

#### find 常见用法

-   常用的几种查找文件方式。

    ```bash
    # 查找所有名称为src的文件夹
    find . -name src -type d
    # 查找所有文件夹路径中包含test的python文件
    find . -path '*/test/*.py' -type f
    # 查找前一天修改的所有文件
    find . -mtime -1
    # 查找所有大小在500k至10M的tar.gz文件
    find . -size +500k -size -10M -name '*.tar.gz'
    ```

-   可以对查找出来的文件执行另一些命令。

    ```bash
    # 删除全部扩展名为.tmp 的文件
    find . -name '*.tmp' -exec rm {} \;
    # 查找全部的 PNG 文件并将其转换为 JPG
    find . -name '*.png' -exec convert {} {}.jpg \;
    ```

### 课后练习

>   1.  阅读 [`man ls`](https://man7.org/linux/man-pages/man1/ls.1.html) ，然后使用`ls` 命令进行如下操作：
>
>       -   所有文件（包括隐藏文件）
>       -   文件打印以人类可以理解的格式输出 (例如，使用454M 而不是 454279954)
>       -   文件以最近访问顺序排序
>       -   以彩色文本显示输出结果
>
>       典型输出如下：
>
>       ```
>        -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
>        drwxr-xr-x   5 user group  160 Jan 14 09:53 .
>        -rw-r--r--   1 user group  514 Jan 14 06:42 bar
>        -rw-r--r--   1 user group 106M Jan 13 12:12 foo
>        drwx------+ 47 user group 1.5K Jan 12 18:08 ..
>       ```

-   所有文件: `-a`
-   人类可以理解的格式输出: `-h`

-   以最近访问顺序排序: `-t`
-   以彩色文本显示输出结果: `--color=auto`

>   2.   编写两个bash函数 `marco` 和 `polo` 执行下面的操作。 每当你执行 `marco` 时，当前的工作目录应当以某种形式保存，当执行 `polo` 时，无论现在处在什么目录下，都应当 `cd` 回到当时执行 `marco` 的目录。 为了方便debug，你可以把代码写在单独的文件 `marco.sh` 中，并通过 `source marco.sh`命令，（重新）加载函数。

```bash
#!/bin/bash
marco() {
    export MARCO=$(pwd)
}
polo() {
    cd "$MARCO"
}
```

>   3.   假设您有一个命令，它很少出错。因此为了在出错时能够对其进行调试，需要花费大量的时间重现错误并捕获输出。 编写一段bash脚本，运行如下的脚本直到它出错，将它的标准输出和标准错误流记录到文件，并在最后输出所有内容。 加分项：报告脚本在失败前共运行了多少次。
>
>        ```bash
>        #!/usr/bin/env bash
>             
>        n=$(( RANDOM % 100 ))
>             
>        if [[ n -eq 42 ]]; then
>            echo "Something went wrong"
>            >&2 echo "The error was using magic numbers"
>            exit 1
>        fi
>             
>        echo "Everything went according to plan"
>        ```
>
>        

```bash
#!/bin/bash
echo > out.log
for((count=0; ;count++)) # 双括号表示运算
do
    ./script.sh &>> out.log # &>> 表示输出流和错误流一起重定向
    if [[ $? -ne 0 ]]; then # 双中括号[[]]，表达式前后要空格
        echo "failed after $count times"
        break
    fi
done
```

 >   4.   本节课我们讲解的 `find` 命令中的 `-exec` 参数非常强大，它可以对我们查找的文件进行操作。但是，如果我们要对所有文件进行操作呢？例如创建一个zip压缩文件？我们已经知道，命令行可以从参数或标准输入接受输入。在用管道连接命令时，我们将标准输出和标准输入连接起来，但是有些命令，例如`tar` 则需要从参数接受输入。这里我们可以使用[`xargs`](https://man7.org/linux/man-pages/man1/xargs.1.html) 命令，它可以使用标准输入中的内容作为参数。 例如 `ls | xargs rm` 会删除当前目录中的所有文件。
 >
 >        您的任务是编写一个命令，它可以递归地查找文件夹中所有的HTML文件，并将它们压缩成zip文件。注意，即使文件名中包含空格，您的命令也应该能够正确执行（提示：查看 `xargs`的参数`-d`，译注：MacOS 上的 `xargs`没有`-d`，[查看这个issue](https://github.com/missing-semester/missing-semester/issues/93)）
 >
 >        如果您使用的是 MacOS，请注意默认的 BSD `find` 与 [GNU coreutils](https://en.wikipedia.org/wiki/List_of_GNU_Core_Utilities_commands) 中的是不一样的。你可以为`find`添加`-print0`选项，并为`xargs`添加`-0`选项。作为 Mac 用户，您需要注意 mac 系统自带的命令行工具和 GNU 中对应的工具是有区别的；如果你想使用 GNU 版本的工具，也可以使用 [brew 来安装](https://formulae.brew.sh/formula/coreutils)。

```bash
find . -type f -name "*.html" | xargs -d '\n'  tar -cvzf html.zip
```
