Windows 系统的 gcc/g++ 可以安装微软官方的 Visual Studio，也可以自己安装 MinGW，由于我平时编写 C/C++ 都是在 VSCode上，所以选择后者。

网上大多数教程都停留在 MinGW 8 的版本，太老了，这是我记录这篇博客的原因。

## 下载 MinGW 安装包

进入 [Downloads - MinGW-w64](https://www.mingw-w64.org/downloads/)，选择 [Mingw-builds](https://www.mingw-w64.org/downloads/#mingw-builds)，会跳转到 github

选择一个 Release，这里我选的是 x86_64-13.2.0-release-posix-seh-ucrt-rt_v11-rev1.7z

解释一下各个文件的不同：

1.   `x64-64` 和 `i686`：`x64-64` 表示 64 位， `i686` 表示 32 位。选 64 就好。
2.   `13.2.0-release`：版本号，选最新的就好。
3.   `win32` 、`posix`、`mcf`：多线程模型，其实没啥区别，选哪个都一样，据说 `posix` 启用了  c++11/c11的多线程功能。
4.   `seh` 和 `draft`：结构化异常处理的东西，`seh` 性能好一些支持 64 位，`draft` 兼容性好一些些，64 和 32 都支持。
5.   `ucrt` 和 `msvcrt` ：`ucrt` 新一点。

## 环境变量

找个解压路径解压，例如 `C:\Program Files`，然后添加环境变量：

此电脑（右键) | 属性 | 高级系统设置 | 高级 | 环境变量 | 系统变量（或者用户变量）| Path（双击）| 新建 | `C:\Program Files\mingw64\bin`

打开终端，输入 `gcc -v` 即可