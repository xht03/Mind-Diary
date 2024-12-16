---
title: Shell简明教程
date: 2024-02-04 10:49:32
tags:
- original
categories:
- The Missing Semester
---

### shell 使用

#### 什么是shell

如今的计算机有着多种多样的交互接口让我们可以进行指令的的输入，如：图像化的用户界面（GUI）、语音输入 、AR/VR 。 这些交互接口可以覆盖 80% 的使用场景，但是它们也从根本上限制了你的操作方式——你不能点击一个不存在的按钮或者是用语音输入一个还没有被录入的指令。为了充分利用计算机的能力，我们不得不回到最根本的方式，使用文字接口：Shell。

shell 通常指的是操作系统提供给用户与内核进行交互的用户接口。**Bourne Again Shell**，简称 **bash**。 这是被最广泛使用的一种 shell，它的语法和其他的 shell 都是类似的。

---

#### shell 大致原理

shell 基于空格分割命令并进行解析，然后执行第一个单词代表的程序，并将后续的单词作为程序可以访问的参数。如果您希望传递的参数中包含空格（例如一个名为 My Photos 的文件夹），您要么用使用单引号，双引号将其包裹起来，要么使用转义符号 `\` 进行处理（`My\ Photos`）。

类似于 Python 或 Ruby，shell 是一个编程环境，所以它具备变量、条件、循环和函数。当你在 shell 中执行命令时，您实际上是在执行一段 shell 可以解释执行的简短代码。如果你要求 shell 执行某个指令，但是该指令并不是 shell 所了解的编程关键字，那么它会去咨询 *环境变量* `$PATH`，它会列出当 shell 接到某条指令时，进行程序搜索的路径。随后它便会在 `$PATH` 中搜索由 `:` 所分割的一系列目录，基于名字搜索该程序。当找到该程序时便执行。

---

#### 基础指令

- 查看时间：`date`

- 打印文本或变量：`echo`

- 查看指定目录下所有文件：`ls`

- 查看命令手册：`man`

  

  *注意：*

  1. *通常，在执行程序时使用 `-h` 或 `--help` 标记可以打印帮助信息，以便了解有哪些可用的标记或选项。*
  2. *如果您想要知道某个命令（程序）怎么用，请试试 `man` 这个程序。它会接受一个程序名作为参数，然后将它的文档（用户手册）展现给您。*

---

#### 在shell中导航

- 当前工作目录：`pwd`

- 切换目录：`cd`

- 查找命令（程序）的路径：`which`

  

  *注意：*

  1. *在 Linux 和 macOS 上，路径使用 `/` 分割，而在Windows上是 `\`。路径 `/` 代表的是系统的根目录，所有的文件夹都包括在这个路径之下；在Windows上每个盘都有一个根目录（例如： `C:\`）。*
  2.  *在 Linux 文件系统中，如果某个路径以 `/` 开头，那么它是一个 **绝对路径**，其他的都是 **相对路径**。相对路径是指相对于当前工作目录的路径。在路径中，`.` 表示的是当前目录，而 `..` 表示上级目录：*
  3. *如果命令在 `PATH` 中找不到，`which` 通常不会输出任何内容。*

---

#### 文件操作

- 创建空文件：`touch`
- 移动文件 或 重命名：`mv`
- 拷贝文件：`cp`
- 新建文件夹：`mkdir`
- 显示文件内容 或 连接文件：`cat`
- 删除文件：`rm`
- 查看文件末尾内容：`tail`

---

#### 重定向

在 shell 中，程序有两个主要的“**流**”：它们的**输入流**和**输出流**。 当程序尝试读取信息时，它们会从输入流中进行读取，当程序打印信息时，它们会将信息输出到输出流中。 通常，一个程序的输入输出流都是你的终端。也就是，你的键盘作为输入，显示器作为输出。 但是，我们也可以重定向这些流！

最简单的重定向是 `< file` 和 `> file`。这两个命令可以将程序的输入输出流分别重定向到文件：

```bash
(Py) keats@OMEN-Yanxu:~/Avalon$ touch hello.txt
(Py) keats@OMEN-Yanxu:~/Avalon$ echo "hello world!" > hello.txt
(Py) keats@OMEN-Yanxu:~/Avalon$ cat hello.txt
hello world!
```

你还可以使用 `>>` 来向一个文件追加内容（*将命令的输出追加到文件的末尾，而不是覆盖文件内容*）。

使用**管道（ *pipes* ）**，我们能够更好的利用文件重定向。 `|` 操作符允许我们将一个程序的输出和另外一个程序的输入连接起来：

```bash
(Py) keats@OMEN-Yanxu:~/Avalon$ ls
ICS  IGEM  PJ  Research  vpn.yaml
(Py) keats@OMEN-Yanxu:~/Avalon$ ls -l | tail -n1
-rw-r--r-- 1 keats keats  504 Jan 26 12:35 vpn.yaml
```

---

#### 根用户

对于大多数的类 Unix 系统，有一类用户是非常特殊的，那就是：**根用户（root user）**。 你应该已经注意到了，在上面的输出结果中，根用户几乎不受任何限制，他可以创建、读取、更新和删除系统中的任何文件。 通常在我们并不会以根用户的身份直接登录系统，因为这样可能会因为某些错误的操作而破坏系统。 取而代之的是我们会在需要的时候使用 `sudo` 命令。顾名思义，它的作用是让您可以以 su（super user 或 root 的简写）的身份执行一些操作。 当您遇到拒绝访问（permission denied）的错误时，通常是因为此时您必须是根用户才能操作。





### shell 脚本

#### 什么是 shell 脚本

shell 脚本是**一系列用 shell 语言编写的命令**，它们按照特定的顺序组织在一个文件中，以执行特定的任务或自动化操作。shell 脚本通常用于执行一系列的命令、条件判断、循环等操作，为用户提供一个方便的方式来批量处理任务。

---

#### 变量赋值

- 为变量赋值：`foo=bar`

- 访问变量中存储的数值： `$foo`

  

  *注意：*

  1. *`foo = bar` （使用空格隔开）是不能正确工作的。因为解释器会调用程序`foo` 并将 `=` 和 `bar`作为参数。 总的来说，在shell脚本中使用空格会起到分割参数的作用，有时候可能会造成混淆，请务必多加检查。*
  2. *Bash中的字符串通过`'` 和 `"`分隔符来定义，但是它们的含义并不相同。以`'`定义的字符串为**原义**字符串，其中的变量不会被**转义**，而 `"`定义的字符串会将变量值进行替换。*

```bash
(Py) keats@OMEN-Yanxu:~/Avalon$ foo=bar
(Py) keats@OMEN-Yanxu:~/Avalon$ echo foo
foo
(Py) keats@OMEN-Yanxu:~/Avalon$ echo $foo
bar
(Py) keats@OMEN-Yanxu:~/Avalon$ echo '$foo'
$foo
(Py) keats@OMEN-Yanxu:~/Avalon$ echo "$foo"
bar
```

---

#### 特殊变量

和其他大多数的编程语言一样，`bash`也支持**`if`, `case`, `while` 和 `for`** 这些控制流关键字。同样地， `bash` 也支持**函数**，它可以接受参数并基于参数进行操作。下面这个函数是一个例子，它会创建一个文件夹并使用`cd`进入该文件夹。

```bash
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```

与其他脚本语言不同，bash使用了很多**特殊的变量**来表示参数、错误代码和相关变量。常见的有：

- `$0` - 脚本名
- `$1` 到 `$9` - 脚本的参数。 `$1` 是第一个参数，依此类推。
- `$@` - 所有参数
- `$#` - 参数个数
- `$?` - 前一个命令的返回值
- `$$` - 当前脚本的进程识别码
- `!!` - 完整的上一条命令，包括参数。*常见应用：当你因为权限不足执行命令失败时，可以使用 `sudo !!`再尝试一次。*
- `$_` - 上一条命令的最后一个参数。*如果你正在使用的是交互式 shell，你可以通过按下 `Esc` 之后键入 . 来获取这个值。*

---

#### 输入输出

一般情况下，每个 Unix/Linux 命令运行时都会打开三个文件：

- **标准输入文件(stdin)**：stdin的文件描述符为0，Unix程序默认从stdin读取数据。
- **标准输出文件(stdout)**：stdout 的文件描述符为1，Unix程序默认向stdout输出数据。
- **标准错误文件(stderr)**：stderr的文件描述符为2，Unix程序会向stderr流中写入错误信息。

默认情况下，`command > file` 将 stdout 重定向到 file，`command < file` 将 stdin 重定向到 file。

- 如果希望 stderr 重定向到 file，可以这样写：

  ```bash
  $ command 2>file	# 2表示标准错误文件(stderr)
  ```

- 如果希望将 stdout 和 stderr 合并后重定向到 file，可以这样写：

  ```bash
  $ command > file 2>&1	# 2>&1 表示将 标准错误2 重定向到与 标准输出1 相同的位置
  ```

- 如果希望对 stdin 和 stdout 都重定向，可以这样写：

  ```bash
  $ command < file1 >file2	# <和>后面是否空格不影响命令执行
  ```

---

#### 返回值（退出码）

命令通常使用 `STDOUT`来返回输出值，使用`STDERR` 来返回错误及错误码，便于脚本以更加友好的方式报告错误。 **返回码或退出状态**是脚本/命令之间交流执行状态的方式。返回值0表示正常执行，其他所有非0的返回值都表示有错误发生。

退出码可以搭配 `&&`（与操作符）和 `||`（或操作符）使用，用来进行条件判断，决定是否执行其他程序。它们都属于[短路运算符](https://en.wikipedia.org/wiki/Short-circuit_evaluation)。同一行的多个命令可以用` ; `分隔。程序 `true` 的返回码永远是`0`，`false` 的返回码永远是`1`。

```bash
(Py) keats@OMEN-Yanxu:~/Avalon$ false || echo "Oops, fail"
Oops, fail
(Py) keats@OMEN-Yanxu:~/Avalon$ true || echo "Will not be printed"
(Py) keats@OMEN-Yanxu:~/Avalon$ true && echo "Things went well"
Things went well
(Py) keats@OMEN-Yanxu:~/Avalon$ false && echo "Will not be printed"
(Py) keats@OMEN-Yanxu:~/Avalon$ false ; echo "This will always run"
This will always run
```

另一个常见的模式是以变量的形式获取一个命令的输出，这可以通过 **命令替换（command substitution）**实现。下面是一个例子：

```bash
(Py) keats@OMEN-Yanxu:~/Avalon$ echo "Starting program at $(date)"
Starting program at Sun Feb  4 16:35:02 CST 2024
```

- 当你通过 `$( CMD )` 这样的方式来执行`CMD` 这个命令时，它的输出结果会替换掉 `$( CMD )` 。

  

  *例如，如果执行 `for file in $(ls)` ，shell首先将调用`ls` ，然后遍历得到的这些返回值。*

还有一个冷门的类似特性是 **进程替换（process substitution）**。这在我们希望返回值通过文件而不是STDIN传递时很有用

-  `<( CMD )` 会执行 `CMD` 并将结果输出到一个临时文件中，并将 `<( CMD )` 替换成临时文件名。。

  

  *例如， `diff <(ls foo) <(ls bar)` 会显示文件夹 `foo` 和 `bar` 中文件的区别。*

---

#### 通配

当执行脚本时，我们经常需要提供形式类似的参数。bash使我们可以轻松的实现这一操作，它可以基于文件扩展名展开表达式。这一技术被称为shell的 **通配（globbing）**。

- **通配符**：当你想要利用通配符进行匹配时，你可以分别使用 `?` 和 `*` 来匹配一个或任意个字符。

  ```bash
  # 对于文件foo, foo1, foo2, foo10 和 bar
  $ rm foo?	# 删除 foo1 和 foo2
  $ rm foo*	# 删除除了`bar`之外的所有文件
  ```

- **花括号`{}`**：当你有一系列的指令，其中包含一段公共子串时，可以用花括号来自动展开这些命令。这在批量移动或转换文件时非常方便。

---

#### shebang

注意，脚本并不一定只有用 bash 写才能在终端里调用。比如说，这是一段 Python 脚本，作用是将输入的参数倒序输出：

```python
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```

内核知道去用 python 解释器而不是 shell 命令来运行这段脚本，是因为脚本的开头第一行的 [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix))。

在 `shebang` 行中使用 `env` 命令是一种很好的实践，它会利用环境变量中的程序来解析该脚本，这样就提高来您的脚本的可移植性。`env` 会利用我们第一节讲座中介绍过的`PATH` 环境变量来进行定位。 例如，此处使用了`env`的shebang看上去时这样的`#!/usr/bin/env python`。

---

#### shell 函数 VS 脚本

初次接触的小伙伴们可能有些疑惑，我们敲进命令行的一条条函数和脚本有什么区别呢？粗略地看，有如下一些不同点：

- 函数只能与shell使用相同的语言，脚本可以使用任意语言。因此在脚本中包含 `shebang` 是很重要的。
- 函数仅在定义时被加载，脚本会在每次被执行时加载。这让函数的加载比脚本略快一些，但每次修改函数定义，都要重新加载一次。
- 函数会在当前的shell环境中执行，脚本会在单独的进程中执行。因此，函数可以对环境变量进行更改，比如改变当前工作目录，脚本则不行。脚本需要使用 `export`将环境变量导出，并将值传递给环境变量。
- 与其他程序语言一样，函数可以提高代码模块性、代码复用性并创建清晰性的结构。shell脚本中往往也会包含它们自己的函数定义。





### shell 工具

#### 查看命令如何使用

`man`命令是手册（manual）的缩写，它提供了命令的用户手册。

---

#### 查找文件

所有的类UNIX系统都包含一个名为 `find`的工具，它是 shell 上用于查找文件的绝佳工具。`find`命令会递归地搜索符合条件的文件。

```bash
# 查找所有名称为src的文件夹
$ find . -name src -type d
# 查找前一天修改的所有文件
$ find . -mtime -1
```

除了列出所寻找的文件之外，`find` 还能对所有查找到的文件进行操作。这能极大地简化一些单调的任务。

```bash
# 删除全部扩展名为.tmp 的文件
$ find . -name '*.tmp' -exec rm {} \;
# 查找全部的 PNG 文件并将其转换为 JPG
$ find . -name '*.png' -exec convert {} {}.jpg \;w
```

---

#### 查看代码

很多类UNIX的系统都提供了`grep`命令，它是用于对输入文本进行匹配的通用工具。

`grep` 有很多选项，这也使它成为一个非常全能的工具。其中我们经常使用的有：

- `-C` ：获取查找结果的上下文（Context）。
- `-v` ：将对结果进行反选（Invert），也就是输出不匹配的结果。
- `-R` ：递归地进入子目录并搜索所有的文本文件。

举例来说， `grep -C 5` 会输出匹配结果前后五行。

---

#### 查找 shell 命令

随着你使用shell的时间越来越久，你可能想要找到之前输入过的某条命令。

- 按**向上的方向键**会显示你使用过的上一条命令，继续按上键则会遍历整个历史记录。

- **`history`** 命令允许您以程序员的方式来访问shell中输入的历史命令。这个命令会在标准输出中打印shell中的里面命令。如果我们要搜索历史记录，则可以利用管道将输出结果传递给 `grep` 进行模式搜索。 例如：`history | grep find` 会打印包含find子串的命令。
- 对于大多数的shell来说，你可以使用 **`Ctrl+R`** 对命令历史记录进行回溯搜索。敲 `Ctrl+R` 后您可以输入子串来进行匹配，查找历史命令行。



### 更多

详见：[missing semester](https://missing.csail.mit.edu/)



