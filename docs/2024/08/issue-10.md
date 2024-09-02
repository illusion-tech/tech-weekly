# 使用Rye来管理Python版本和依赖

## 什么是Rye

Rye 是一个用于管理 Python 版本和依赖项的工具，它基于 Rust 构建，旨在提供一种简单、高效的方式来管理 Python 项目。Rye 的目标是成为 Python 开发者的首选工具，提供类似于 npm 的体验。

## 安装Rye

在安装 Rye 之前，需要先安装 Rust。Rust 的安装方法可以参考官方文档：https://www.rust-lang.org/tools/install

安装 Rust 后，可以使用以下命令来安装 Rye：

```bash
curl -sSf https://rye.astral.sh/get | bash
```
按照步骤安装成功后，执行source
```bash
source "$HOME/.rye/env"
```

## 添加命令自动完成

```bash
mkdir -p ~/.local/share/bash-completion/completions
rye self completion > ~/.local/share/bash-completion/completions/rye.bash
```

## 使用Rye

> **注意**：这里只介绍Rye的基础用法，如果你需要更深入的了解，可以参考官方文档：https://rye.astral.sh/guide/


### 创建一个新的Python项目

使用 Rye 创建一个新的 Python 项目非常简单，只需要运行以下命令：

```bash
rye init my_project # 创建项目
cd my_project       # 进入项目目录
rye sync            # 同步项目依赖
```

这将会创建一个新的 Python 项目，并自动安装所需的依赖项。

### 选择Python版本

使用 Rye 安装 Python 版本也非常简单，只需要运行以下命令：

```bash
rye pin 3.10
```

这将会安装 Python 3.10 版本，并将其设置为当前项目的默认版本。

### 管理Python依赖项
#### 添加依赖项
使用 Rye 管理 Python 依赖项也非常简单，只需要运行以下命令：

```bash
rye add requests
```

这将会安装 requests 库，并将其添加到项目的依赖项中。

#### 删除依赖项

```bash
rye remove requests
```

这将会删除 requests 库，并将其从项目的依赖项中移除。

#### 列出依赖项

```bash
rye list
```

这将会更新项目的所有依赖项到最新版本。

### 运行Python项目

使用 Rye 运行 Python 项目也非常简单，只需要运行以下命令：

```bash
rye init --script my_project # 生成执行脚本
rye sync
rye run my_project # 运行项目
```

这将会运行当前项目的默认 Python 版本，并执行项目的入口文件。

> 我亲自测试认为Rye并没有像官方文档说的那么直接创建项目入口文件，在我的研究下发现，手动写项目入口会比较好用一点，如果官方看到希望能把这部分内容完善一下。

在`pyproject.toml`文件中添加以下内容来指定项目入口文件：

```toml
[project.scripts]
hello = "my_project:main"
```

然后在`src/my_project/__init__.py`中添加以下内容

```python
def hello() -> str:
    return "Hello from rye-hello!"

def main():
    print(hello())
```

此时运行
```bash
rye run hello
```
即可看到输出
```bash
Hello from rye-hello!
```

> 从现在就能看到Rye在管理Python项目上的优势是多么出色，尽管还存在一些问题，但我相信最终官方会把它做得更好。由于时间精力有限，我无法一一测试，如果有问题，欢迎留言讨论，或者者在官方文档中寻找答案。

## 总结

Rye 是一个强大的 Python 版本和依赖项管理工具，它基于 Rust 构建，旨在提供一种简单、高效的方式来管理 Python 项目。使用 Rye，你可以轻松地安装和管理 Python 版本和依赖项，从而提高你的 Python 开发效率。

- 仓库地址：https://github.com/xsxz01/rye_hello