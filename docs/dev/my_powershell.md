## Windows Terminal 配置

>  在微软商店安装 [PowerShell](https://apps.microsoft.com/detail/9MZ1SNWT0N5D?hl=zh-cn&gl=CN) 和 [Windows Terminal ](https://apps.microsoft.com/detail/9N0DX20HK701?hl=zh-cn&gl=cn)

打开终端按照如下设置。

### 启动

|      设置项      |        设置         |
| :--------------: | :-----------------: |
|   默认配置文件   |     PowerShell      |
| 默认终端应用程序 |    Windows 终端     |
|     启动大小     | 列：95 <br />行：30 |
|     启动参数     |    在启动时居中     |

### 外观

|          设置项          | 设置 |
| :----------------------: | :--: |
| 在选项卡中使用亚克力材料 |  开  |

### 配色方案

使用 `Snazzy`,参考 [Richienb/windows-terminal-snazzy: Elegant Windows Terminal theme with bright colors (github.com)](https://github.com/Richienb/windows-terminal-snazzy?tab=readme-ov-file) 的安装步骤：

1. 在设置中打开JSON文件。

2. 在 `schemes` 中添加如下：

```json
        {
            "background": "#282A36",
            "black": "#282A36",
            "blue": "#57C7FF",
            "brightBlack": "#686868",
            "brightBlue": "#57C7FF",
            "brightCyan": "#9AEDFE",
            "brightGreen": "#5AF78E",
            "brightPurple": "#FF6AC1",
            "brightRed": "#FF5C57",
            "brightWhite": "#EFF0EB",
            "brightYellow": "#F3F99D",
            "cursorColor": "#97979B",
            "cyan": "#9AEDFE",
            "foreground": "#EFF0EB",
            "green": "#5AF78E",
            "name": "Snazzy",
            "purple": "#FF6AC1",
            "red": "#FF5C57",
            "selectionBackground": "#3E404A",
            "white": "#F1F1F0",
            "yellow": "#F3F99D"
        },
```

3. 在 **设置-配色方案** 中选择 `Snazzy` 主题。

### 操作

快捷键根据自己需求设置。

### 配置文件-默认值-外观

|     设置项     |                             设置                             |
| :------------: | :----------------------------------------------------------: |
|    配色方案    |                            Snazzy                            |
|      字体      | FiraCode Nerd Font Mono<br />[下载链接](https://www.nerdfonts.com/font-downloads) |
|      字号      |                              14                              |
|  背景不透明度  |                             90%                              |
| 启用亚克力材料 |                              开                              |

### 配置文件-PowerShell

调换一下顺序

1. 在设置中打开JSON文件。
2. 在 `profiles` - `list` 中，将 `PowerShell` 的部分移动到最前面。

---

## 更改 PowerShell 语法颜色

> 默认情况下，powershell 中某些语法颜色很奇怪，例如 `git -v` 的 `-v` 显示的颜色非常浅，`git` 的颜色也是黄色，这和 zsh 不太一样。

### PowerShell 配置文件路径

zsh 在启动时会执行 `~/.zshrc` 中的所有语句，用于 zsh 。 同样的，powershell 也有类似的文件，在终端中输入以下命令：

```powershell
echo $PROFILE
```

输出为：

```pow
~\Documents\PowerShell\Microsoft.PowerShell_profile.ps1
```

变量 `$PROFILE` 存储的是当前用户当前主机的 powershell 配置文件，等价于 `$PROFILE.CurrentUserCurrentHost`，`$PROFILE` 具有以下值：

- 当前用户，当前主机 - `$PROFILE`
- 当前用户，当前主机 - `$PROFILE.CurrentUserCurrentHost`
- 当前用户，所有主机 - `$PROFILE.CurrentUserAllHosts`
- 所有用户，当前主机 - `$PROFILE.AllUsersCurrentHost`
- 所有用户，所有主机 - `$PROFILE.AllUsersAllHosts`

### 使用 PSReadLine 修改 PowerShell 字体颜色

PSReadLine 是 powershell 自带的命令行自定义模块，提供以下功能：

- **命令行语法着色**
- 语法错误的直观指示
- 更好的多行体验（编辑和历史记录）
- 可自定义的键绑定
- Cmd 和 Emacs 模式
- 许多配置选项
- Bash 风格补全（Cmd 模式下为可选，Emacs 模式下为默认）
- Emacs yank/kill-ring
- 基于 PowerShell 标记的“单词”移动和删除
- 预测 IntelliSense
- 在控制台中动态显示帮助，而不会丢失在命令行中的位置

第一个功能就是修改命令行语法颜色，下面进行修改

首先输入以下命令查看 PSReadLine 的默认设置：

```powershell
Get-PSReadLineOption
```

可以看到最后一部分是带有颜色的字体，有关字体颜色以及输出的相关知识，可以参考 [PowerShell : 如何设置输出颜色，Format-Color让黑乎乎的窗口丰富起来_powershell字体不变色-CSDN博客](https://blog.csdn.net/gjmjack/article/details/121179867) 。

下面修改语法着色的部分，参考官方的 [Set-PSReadLineOption (PSReadLine) - PowerShell | Microsoft Learn](https://learn.microsoft.com/zh-cn/powershell/module/psreadline/set-psreadlineoption?view=powershell-7.4) 。

首先打开配置文件 

```powershell
code $PROFILE
```

添加语法颜色修改的配置，使其更接近 zsh 的语法着色：

```powershell
# $PROFILE
Set-PSReadLineOption -Colors @{
    Command     = 'Green'
    Parameter   = 'White'
    String      = 'Yellow'
    Variable    = 'White'
}
```

右侧的值可以选择以下之一（应该也支持 ANSI 和 RBG，未测试）

- `Black`
- `DarkBlue`
- `DarkGreen`
- `DarkCyan`
- `DarkRed`
- `DarkMagenta`
- `DarkYellow`
- `Gray`
- `DarkGray`
- `Blue`
- `Green`
- `Cyan`
- `Red`
- `Magenta`
- `Yellow`
- `White`

---

## Git 命令补全、状态显示

> 目前命令行针对 `git` 相关的命令不能补全，也不能显示当前分支。

### 安装 Git

下载链接：[Git - Downloading Package (git-scm.com)](https://git-scm.com/download/win)，开始安装。

安装选项中，最好勾上 “Add a Git Bash Profile to Windows Terminal”，因为以后有可能在终端打开 Git Bash。

其他的选项根据喜好选择。

### 安装 Posh-Git

Posh-Git 提供针对 `git` 命令的补全，以及显示当前分支等功能。

输入以下命令开始安装：

```powershell
Install-Module posh-git
```

安装完成后，在 `$PROFILE` 中添加：

```powershell
# $PROFILE
Import-Module Posh-Git
```

随后新建一个 powershell，就支持命令补全以及状态显示了。（目前状态显示的字体有点问题，等一下会修改。）

## 使用 oh-my-posh 美化命令提示符

> 目前的命令提示符还不够美观，并且缺乏命令执行时间等功能。微软官方推荐使用 `oh-my-posh` 来美化命令提示符以及拓展相关的功能。

### 安装 oh-my-posh

微软商店直接安装：[oh-my-posh - Microsoft Apps](https://apps.microsoft.com/detail/XP8K0HKJFRXGCK?hl=zh-cn&gl=CN)

### 选择主题：

oh-my-posh 提供许多现成的主题，具体可以参考 [Themes | Oh My Posh](https://ohmyposh.dev/docs/themes)、

不同的主题有不同的美化效果，还提供一些额外的功能，例如：

- 显示当前路径
- 显示 git 分支、版本号等

- 显示当前用户名、主机名
- 显示当前时间
- 显示电脑内存
- 显示上一条命令执行的时间

选择喜欢的主题，并记下主题名。

### 设置主题

> 我在 Unix 系统下的 zsh 使用的是 [pure](https://github.com/sindresorhus/pure#oh-my-zsh) 主题，因此这里也选择 `pure` 主题。

在 `$PROFILE` 的第一行添加：

```powershell
# $PROFILE
oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH/pure.omp.json" | Invoke-Expression
```

将 `pure` 换成其他的主题名即可。

设置完成后，新建一个终端，命令提示符样式就会发生改变。

### 修改主题

oh-my-posh 的 pure 主题和 zsh 的 pure 主题并不是完全相似的，具体来说有以下 2 点不同：

1. oh-my-posh 的 pure 主题默认显示用户名，但 zsh 的 pure 主题在本地连接的情况下并不会显示用户名。
2. oh-my-posh 的 pure 主题某些部分的字体颜色和 zsh 的 pure 主题不一致。

修改主题之前，需要复制原有主题的一个副本，执行以下命令（这里我将其复制到和 `$PROFILE` 的同一目录下）：

```powershell
mv $env:POSH_THEMES_PATH/pure.omp.json ~/Documents/PowerShell/my_pure.omp.json
```

修改 `my_pure.omp.json` 就能修改主题了，主题可修改的内容很多，参考 [Introduction | Oh My Posh](https://ohmyposh.dev/docs)。

针对我的 2 个需求，修改方法如下：

#### 去除用户名显示

这个很简单，直接删掉有 `segments` 里带有 `{{ .UserName}}` 的那一段就好了。

#### 修改部分字体颜色

使用 vscode 打开配置文件，带有 `foreground` 的标签就是字体的颜色，直接修改就好。

字体的颜色是 RGB 格式，比较麻烦的是要如何知道修改成什么颜色，例如我想要改成和 zsh 的 pure 主题一样的字体颜色，就必须获得 zsh 的 pure 主题的颜色值。解决的办法有两个，一是直接查阅 zsh 的 pure 主题的源码，直接在 github 上搜索。二是直接捕获屏幕的像素点的值，我用的是 `PowerToys` 提供的颜色选取工具 [PowerToys适用于 Windows 的 ColorPicker 实用工具 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/powertoys/color-picker)。用法也很简单，首先在 windows 终端 ssh 到一台配好了 zsh 和 pure 主题的 Unix 系统上（云服务器、虚拟机、WSL 都可以） ，然后使用 `CTRL+滚轮` 放大终端，最后用快捷键直接捕捉屏幕像素点的 RGB 值就可以了，简单快捷。

最终我修改了 pure 主题这些部分的字体颜色：

|     type      |  修改前  |  修改后  |
| :-----------: | :------: | :------: |
|     path      | \#81A1C1 | \#57C7FF |
| executiontime | \#A3BE8C | \#F3F99D |
|    status     | #B48EAD  | \#FF6AC1 |

改了这三个就和 zsh 的 pure 主题差不多了。

## 让 PowerShell 更加接近 zsh

> 配置一些插件让 powershell 更加美观、更加好用。此外，powershell 的命令名称和 zsh 不太一致，可以通过添加别名实现。

### z ：自动跳转

类似于 zsh 的自动跳转插件，输入以下命令安装：

```powershell
Install-Module z
```

然后在 `$PROFILE` 添加：

```powershell
# $PROFILE
Import-Module z
```

### Terminal-Icons：终端图标

在 `ls` 的时候文件前附加图标显示（不然的话默认情况下文件夹的高亮显示不太好看），输入以下命令安装：

```powershell
Install-Module Terminal-Icons
```

然后在 `$PROFILE` 添加：

```powershell
# $PROFILE
Import-Module Terminal-Icons
```

### 使用 Set-Alias 给命令起别名

#### open 命令

```powershell
Set-Alias open ii
```

### 使用函数创建类似的命令

#### whereis命令

```powershell
# 既能查文件也能查命令
function whereis ($command) {
    Get-Command -Name $command -ErrorAction SilentlyContinue | 
    Select-Object -ExpandProperty Path -ErrorAction SilentlyContinue
}
```

## 适配 vscode 终端

> 解决 vscode 终端缺失字体的问题

在设置中搜索 `Font`，将 `Terminal > Integrated: Font Family` 设置为 `FiraCode Nerd Font` 即可。

## 我的最终配置文件

### `$PROFILE` 文件

```powershell
# oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH/clean-detailed.omp.json" | Invoke-Expression
oh-my-posh init pwsh --config "~/Documents/PowerShell/my_pure.omp.json" | Invoke-Expression
Set-PSReadLineOption -Colors @{
    Command     = 'Green'
    Parameter   = 'White'
    String      = 'Yellow'
    Variable    = 'White'
}

Import-Module Posh-Git
Import-Module Terminal-Icons
Import-Module z

Set-Alias open ii

function clash {
    $Env:http_proxy = "http://127.0.0.1:7890"
    $Env:https_proxy = "http://127.0.0.1:7890"
}
function clash-off {
    $Env:http_proxy = $Env:https_proxy = $null
}

# 既能查文件也能查命令
function whereis ($command) {
    Get-Command -Name $command -ErrorAction SilentlyContinue | 
    Select-Object -ExpandProperty Path -ErrorAction SilentlyContinue
}
```

### `my_pure.omp.json` 文件

```json
{
  "$schema": "https://raw.githubusercontent.com/JanDeDobbeleer/oh-my-posh/main/themes/schema.json",
  "version": 2,
  "blocks": [
    {
      "type": "prompt",
      "alignment": "left",
      "newline": true,
      "segments": [
        {
          "foreground": "#57C7FF",
          "properties": {
            "style": "full"
          },
          "style": "plain",
          "template": "{{ .Path }} ",
          "type": "path"
        }
      ]
    },
    {
      "type": "prompt",
      "alignment": "left",
      "segments": [
        {
          "foreground": "#6C6C6C",
          "properties": {
            "branch_ahead_icon": "<#88C0D0>\u21e1 </>",
            "branch_behind_icon": "<#88C0D0>\u21e3 </>",
            "branch_icon": "",
            "fetch_stash_count": true,
            "fetch_status": true,
            "fetch_upstream_icon": true,
            "github_icon": ""
          },
          "style": "plain",
          "template": "{{ .UpstreamIcon }}{{ .HEAD }}{{if .BranchStatus }} {{ .BranchStatus }}{{ end }}{{ if .Working.Changed }}<#FFAFD7>*</>{{ .Working.String }}{{ end }}{{ if and (.Working.Changed) (.Staging.Changed) }} |{{ end }}{{ if .Staging.Changed }} \uf046 {{ .Staging.String }}{{ end }}{{ if gt .StashCount 0 }} \ueb4b {{ .StashCount }}{{ end }} ",
          "type": "git"
        }
      ]
    },
    {
      "alignment": "left",
      "segments": [
        {
          "foreground": "#F3F99D",
          "properties": {
            "style": "austin"
          },
          "style": "plain",
          "template": " {{ .FormattedMs }} ",
          "type": "executiontime"
        }
      ],
      "type": "prompt"
    },
    {
      "type": "prompt",
      "alignment": "left",
      "newline": true,
      "segments": [
        {
          "foreground": "#FF6AC1",
          "foreground_templates": [
            "{{ if gt .Code 0 }}#BF616A{{ end }}"
          ],
          "properties": {
            "always_enabled": true
          },
          "style": "plain",
          "template": "\u276f ",
          "type": "status"
        }
      ]
    }
  ],
  "console_title_template": "{{if .Root}}(Admin){{end}} {{.PWD}}"
}
```