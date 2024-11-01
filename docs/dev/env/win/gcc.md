Windows 系统的 gcc/g++ 可以安装微软官方的 Visual Studio，也可以自己安装 MinGW，由于我平时编写 C/C++ 都是在 VSCode上，所以选择后者。网上大多数教程都停留在 MinGW 8 的版本，太老了。本篇博客直接安装最新的 MinGW。

## MinGW-W64-builds 和 LLVM

进入 **[Downloads - MinGW-w64](https://www.mingw-w64.org/downloads/)**，可以看到支持 Windows 的只有 Cygwin、LLVM-MinGW、MinGW-W64-builds、MSYS2、w64devkit、WinLibs.com。这里我主要推荐两个：LLVM-MinGW 和 MinGW-W64-builds

选择 LLVM 主要有两个原因：

1.   更新速度快，笔者写下本文时 LLVM 已经更新到了 LLVM 19.1.0，是所有预编译版本中最新的。
2.   同时支持 Windows, Linux, macOS。

但是 LLVM 默认是没有 gdb 的，比较麻烦。

而  MinGW-W64-builds 自带 gdb，轻松省事。总的来说，如果是 MacOS 或者 Ubuntu 我都推荐 LLVM，因为 MacOS 有 Homebrew，Ubuntu 有 apt，安装 gdb 都很轻松。而如果是 Windows 的话还是用 MinGW-W64-builds。

## 下载 MinGW 安装包

点击 **[LLVM-MinGW](https://www.mingw-w64.org/downloads/#llvm-mingw)**，进入 **[Installation: GitHub](https://github.com/mstorsjo/llvm-mingw/releases)**

选择 Release，这里我选的是 x86_64-14.2.0-release-win32-seh-ucrt-rt_v12-rev0.7z

解释一下`ucrt` 和 `msvcrt` 的选择，其实就只是运行时库的区别，msvcrt 兼容比较老的版本，而 ucrt 比较新，所以用 ucrt 就好。

win32 和 posix 是线程模型的区别 Windows 用 win32 就好。

## 配置环境变量

找个解压路径解压，例如 `C:\Program Files`，然后添加环境变量：

键盘 **Win** |输入 [环境变量] | 编辑系统环境变量|系统变量（或用户变量）| Path（双击）| 新建 | `C:\Program Files\mingw64\bin`

打开终端，输入 `gcc -v` 或者 `g++ -v`，显示内容表示安装成功。

