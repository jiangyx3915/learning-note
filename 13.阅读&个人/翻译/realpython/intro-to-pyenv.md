> 本文为译文，原文链接 [https://realpython.com/intro-to-pyenv/](https://realpython.com/intro-to-pyenv/) 
> 本人博客: [编程禅师](http://blog.jiangyixin.top)

你是否曾想为一个支持多个Python版本的项目做出贡献，但不确定如何轻松[测试](https://realpython.com/python-testing/)所有的版本？你是否曾对最新和最好的Python版本感到好奇？也许你想尝试这些新功能，但你担心会破坏你的开发环境。幸运的是，使用`pyenv`管理多个Python版本可以解决这个难题。

本文将为你提供一个很好的介绍，如何最大限度地利用你在项目上工作的时间，并最大限度地减少探索合适Python版本的时间。

**在本文中，你将学习如何：**

1. 安装多个版本的Python
2. 安装最新的Python开发版本
3. 在已安装的版本之间切换
4. 通过`pyenv`使用虚拟环境 
5. 自动激活不同的Python版本和虚拟环境

# 为何使用pyenv？

`pyenv`是管理多个Python版本的绝佳工具。即使你已经在系统上安装了Python，也值得安装`pyenv`，以便你可以轻松地尝试新的语言功能或帮助你创建基于不同Python版本的项目。

## 为什么不使用System Python？

“System Python”是安装在操作系统上的Python。如果你使用的是Mac或Linux，那么默认情况下，当在终端输入`python`时，你将获得一个Python REPL。

那么，为什么不使用它呢？一种看法是这个Python真的*属于*操作系统。毕竟，它是操作系统自带的。当你运行时`which` 命令时会反映出来：

```shell
$ which python
/usr/bin/python
```

在这里，所有用户都可以看到`python`的位置是`/usr/bin/python`。但这可能不是你想要的Python版本：

```python
$ python -V
Pyhton 2.7.12
```

要在系统Python中安装Python包，你必须运行`sudo pip install`。然而这会带来一个问题，你正在全局安装Python包，但如果另一个用户希望安装稍微较旧版本的软件包，则会产生冲突。

安装同一个包的多个版本的问题往往会蔓延到你身上并在你最不期望的时候发生。这个问题通常出现于是一个流行且稳定的包突然在你的系统上出现问题。经过数小时的故障排除和谷歌搜索后，你可能会发现你安装了错误版本的依赖项，这会浪费你一天的时间。

即使安装了新的Python版本在`/usr/local/bin/python3`，你仍然不安全。你将遇到上述相同的权限和灵活性问题。

此外，你对操作系统上安装的Python版本并没有多少控制权。如果你想使用Python中的最新功能，而你又在Ubuntu上，但是它的默认版本可能太旧，这意味着你只能等待新的操作系统出来。

最后，一些操作系统实际上使用了固定的Python版本进行工作。就拿`yum`举例，它大量使用Python来完成其工作。如果你安装新版本的Python并且不小心将其安装到你的用户空间，则可能会严重损害你使用操作系统的能力。

## 那包管理器怎么样?

下一个合乎逻辑的选项则是包管理器。程序例如`apt`，`yum`，`brew`，或者`port`是典型的下一个选项。毕竟，这是你将大多数软件包安装到系统的方法。不幸的是，使用包管理器会发现一些相同的问题。

默认情况下，程序包管理器倾向于将其程序包安装到全局系统空间而不是用户空间。同样，这些系统级软件包会污染你的开发环境，并且难以与其他人共享工作区。

同样再一次，你仍然无法控制安装的Python版本。确实有些存储库给你提供了更多选择，但默认情况下，你要查看特定供应商在任何特定日期所使用的Python版本。

即使你从包管理器安装Python，也要考虑如果你正在编写包的代码在Python 3.4 - 3.7上运行和测试会发生什么。

当输入`python3`时系统会发生什么？你将如何在不同版本之间快速切换？你当然可以手动安装多个不同版本，但这样很乏味且容易出错。如果你想要使用PyPy，Jython或Miniconda，那使用包管理工具可能会更糟。

考虑到这些限制因素，让我们回顾一下可以轻松灵活地安装和管理Python版本的标准：

1. 在你的用户空间中安装Python
2. 安装多个版本的Python
3. 指定所需的确切Python版本
4. 在已安装的版本之间切换

`pyenv` 能让你做这些所有事情或者更多。

***

# 安装pyenv

在安装`pyenv`之前，你需要安装一些特定于操作系统的依赖项。这些依赖项主要是用C编写的开发公用程序，并且是必需的，因为`pyenv`通过从源代码构建来安装Python。有关依赖项的更详细的分类和说明，你可以查看[官方文档。](https://devguide.python.org/setup/#build-dependencies)在本教程中，你将看到安装这些依赖项的最常用方法。

> **注意：** `pyenv`最初并不支持Windows。然而，最近变得活跃的[pyenv-win](https://github.com/pyenv-win/pyenv-win)项目似乎有一些基本的支持。如果你使用Windows，请随时查看。

## 构建依赖项

`pyenv`从源代码构建Python，这意味着你需要安装依赖项才能实际使用`pyenv`。构建依赖项因平台而异。如果你使用的是**Ubuntu / Debian**并且想要安装构建依赖项，则可以使用以下命令：

```shell
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev \ 
libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev \
libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python-openssl
```

这使用[Apt](https://wiki.debian.org/Apt)来安装所有构建依赖项。

如果你使用**Fedora / CentOS / RHEL**，则可以使用`yum`安装构建依赖项：

```shell
sudo yum install zlib-devel bzip2 bzip2-devel readline-devel sqlite \
sqlite-devel openssl-devel xz xz-devel libffi-devel
```

此命令将使用安装Python的所有构建依赖项`yum`。

**macOS**用户可以使用以下命令：

```shell
brew install readline xz
```

此命令依赖于[Homebrew](https://brew.sh/)并为macOS用户安装少量依赖项。

如果你改为使用**openSUSE，**那么你将运行以下命令：

```shell
zypper in zlib-devel bzip2 libbz2-devel libffi-devel \
libopenssl-devel readline-devel sqlite3 sqlite3-devel xz xz-devel
```

此命令再次为你的系统安装所有Python构建依赖项。

最后，对于**Alpine**用户，你可以使用：

```shell
apk add libffi-dev ncurses-dev openssl-dev readline-dev \
tk-dev xz-dev zlib-dev
```

此命令[`apk`](https://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management)用作包管理器，并将在Alpine上安装Python的所有构建依赖项。

