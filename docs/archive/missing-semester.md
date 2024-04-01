课程讲义：[计算机教育中缺失的一课 · the missing semester of your cs education (missing-semester-cn.github.io)](https://missing-semester-cn.github.io/)

视频链接（中文字幕）：[刘黑黑a的个人空间-刘黑黑a个人主页-哔哩哔哩视频 (bilibili.com)](https://space.bilibili.com/518734451)

## 第 1 讲 - 课程概览与 shell

本讲介绍了 `shell` 的基础知识以及常见的命令。

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

