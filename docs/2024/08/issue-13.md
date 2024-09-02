# tauri的command拆分模块

## 前言

上期说到，tauri的command拆分模块是更合理的一种方式，如果全部都将内容写到`src-tauri/src/main.rs`中，会导致代码难以维护，而且会显得非常臃肿，太难看了。

作为一个程序员，把代码写的清晰整洁，可维护性高，是基本的修养，且对于自己而言也是非常有意义的。

此前未在官方文档中发现这部分内容，可能官方是想让我们自己探索吧。但是rust毕竟是有点难度的，不像其他编程语言那样学了基础的部分就可以自由组织代码，rust开发的过程就是如履薄冰，每一步都有可能出问题，而且错误提示还踢皮球，没有点耐心是搞不定的。因此在这里，我分享下我的拆分模块过程。

## 1. 规划目录结构
首先，我们需要规划下目录结构，这里我参考了官方的目录结构，如下：
```
src
├── main.rs
├── app
│   ├── main.rs
│   ├── lib.rs
│   ├── commands
│   │   ├── greet.rs
│   │   └── ...
│   ├── utils
│   │   ├── utils.rs
│   │   └── ...
│   ├── ...
└── ...
```
其中，`main.rs`是入口文件,`lib.rs`是应用模块的入口文件，`commands`目录存放命令模块，`utils`目录存放工具模块，`app`目录存放应用模块。

## 2. 编写command模块
在`commands`目录下，我们可以创建多个命令模块，例如`hello.rs`，`add.rs`等，每个模块对应一个命令，例如：
```rust
// hello.rs
#[tauri::command]
pub fn hello(name: String) -> String {
    format!("Hello, {}!", name)
}
```
在`lib.rs`中申明该模块
```rust
// lib.rs
pub mod commands;
```
这样，我们就可以在`main.rs`中使用`hello`命令了。

> **注意**:模块中的command函数必须使用pub标记导出，然后在`lib.rs`中申明模块，才可以在`main.rs`中使用。而且每个模块中的command名称必须是唯一的，也就是说两个相同名称的command可以在不同的模块中存在，但是绝不能在同一个模块中存在。

## 3. 注册command
在`main.rs`中，我们可以使用`tauri::command`宏来注册命令，例如：
```rust
// main.rs
fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![greet, hello])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```
这样，我们就创建了一个`hello`命令，可以在前端调用`invoke('hello', { name: 'John' })`来调用它。

最后的目录结构如下：
```
src
├── main.rs
├── app
│   ├── main.rs
│   ├── lib.rs
│   ├── commands
│   │   ├── greet.rs
│   │   ├── hello.rs
│   │   └── ...
│   ├── utils
│   │   ├── utils.rs
│   │   └── ...
│   ├── ...
└── ...
```
这样，我们就完成了tauri的command拆分模块的创建。
