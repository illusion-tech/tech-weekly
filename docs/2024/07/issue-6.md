# 如何在 Typescript 项目中使用 Rust

通过WASM,实现在typescript项目中使用rust.

## 1. 创建 Rust Wasm 包

首先，要创建一个rust项目，然后打开它。

```sh
  mkdir wasm-calc && \
  cd wasm-calc && \
  cargo new --lib rust-calc
```

用你喜欢的IDE打开这个项目，然后把下面的代码放到`lib.rs`中，


```rust
pub fn sum(left: i32, right: i32) -> i32 {
    left + right
}

pub fn subtract(left: i32, right: i32) -> i32 {
    left - right
}

pub fn multiply(left: i32, right: i32) -> i32 {
    left * right
}

pub fn divide(left: i32, right: i32) -> i32 {
    left / right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        let result = sum(2, 2);
        assert_eq!(result, 4);
    }

    #[test]
    fn test_subtract() {
        let result = subtract(2, 2);
        assert_eq!(result, 0);
    }

    #[test]
    fn test_multiply() {
        let result = multiply(2, 2);
        assert_eq!(result, 4);
    }

    #[test]
    fn test_divide() {
        let result = divide(2, 2);
        assert_eq!(result, 1);
    }
}
```

然后我们准备通过[wasm-bindgen](https://github.com/rustwasm/wasm-bindgen)这个包来导出来作为一个 WebAssembly (Wasm)包。（这个包提供了生成rust到JavaScript绑定的能力）

```sh
  cargo add wasm-bindgen
```

修改 `Cargo.toml` 中的 `crate-type`,如果没有你就加上，就像下面这样。这个指令可以让rust生成对应的wasm包。_[参考](https://users.rust-lang.org/t/why-do-i-need-to-set-the-crate-type-to-cdylib-to-build-a-wasm-binary/93247)_

```toml
[lib]
crate-type = ["cdylib"]
```
在 `lib.rs` 中使用 `wasm_bindgen`:

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn sum(left: i32, right: i32) -> i32 {
    left + right
}

#[wasm_bindgen]
pub fn subtract(left: i32, right: i32) -> i32 {
    left - right
}

#[wasm_bindgen]
pub fn multiply(left: i32, right: i32) -> i32 {
    left * right
}

#[wasm_bindgen]
pub fn divide(left: i32, right: i32) -> i32 {
    left / right
}
```

然后使用 [wasm-pack](https://github.com/rustwasm/wasm-pack) 来构建包，这个工具可以构建不同JavaScript环境的包。

```sh
  wasm-pack build --out-dir target/pkg-node --target nodejs
```

## 2. 在 Node.js 中使用 WASM 包

创建一个新的 NodeJs 项目:

```sh
  cd ../ && \
  mkdir node-rust-calc && \
  cd node-rust-calc && \
  npm init -y && \
  npm add typescript -D && \
  npx tsc --init && \
  touch index.ts
```

把rust包作为一个依赖导入

```sh
  npm add ../rust-calc/target/pkg-node
```

在 index.ts 中导入 rust 写的函数

```typescript
import { sum, divide, multiply, subtract } from "rust-calc";

console.log("1 + 2: ", sum(1, 2)); // 3
console.log("1 - 2: ", subtract(1, 2)); // -1
console.log("1 * 2: ", multiply(1, 2)); // 2
console.log("1 / 2: ", divide(1, 2)); // 0
```

运行NodeJs程序

```sh
npx tsc && node index.js
```

## 3. 在 Vanilla 中使用 WASM 开发 Web

构建目标平台（Web）的WASM包

```sh
cd ../rust-calc
wasm-pack build --out-dir target/pkg-web --target web
```

> 这里构建的web包跟NodeJs包是不同的，需要区分。

创建一个Web的项目（vanilla项目）

```sh
  cd ../ && \
  mkdir vanilla-js-rust-calc && \
  cd vanilla-js-rust-calc && \
  touch index.html
```

在新建的  `index.html` 文件中加入以下内容

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Vanilla JS Rust Calc</title>
  </head>
  <body>
    <h1>Vanilla JS Rust Calc</h1>
    <p>Open the console to see the output</p>
    <script type="module">
      import init, {
        sum,
        subtract,
        multiply,
        divide,
      } from "../rust-calc/target/pkg-web/rust_calc.js";

      async function run() {
        await init();
        console.log("1 + 2: ", sum(1, 2)); // 3
        console.log("1 - 2: ", subtract(1, 2)); // -1
        console.log("1 * 2: ", multiply(1, 2)); // 2
        console.log("1 / 2: ", divide(1, 2)); // 0 As the rust function is working with integers, we gonna receive 0 instead of 0.5
      }

      run();
    </script>
  </body>
</html>
```

保存 `index.html` ，然后使用一个简单的服务器启动就好，如果你使用的是vscode,那么可以用liveserver，作者这里使用的是 [miniserve](https://github.com/svenstaro/miniserve)
> 不可以直接打开`index.html`，因为WASM需要服务器加载，如果直接打开就会跨域。

```sh
  cd ../
  miniserve . --index "vanilla-js-rust-calc/index.html" -p 8080
```

Open [http://localhost:8080](http://localhost:8080) in your browser and check the console.
在浏览器中打开 [http://localhost:8080](http://localhost:8080) ，然后打开 `开发者工具(F12)` ，选中控制台选项，就可以看到效果了。


## 4. 在 Next.js 中使用 WASM

创建一个Next.js项目

```sh
  npx create-next-app@14.2.4 nextjs-rust-calc --use-npm
```

然后添加rust包

```sh
  cd nextjs-rust-calc && \
  npm add ../rust-calc/target/pkg-web
```


在`page.tsx`中写入以下内容

```tsx
import { sum, subtract, multiply, divide } from "rust-calc";

export default function Home() {
  console.log("1 + 2: ", sum(1, 2)); // 3
  console.log("1 - 2: ", subtract(1, 2)); // -1
  console.log("1 * 2: ", multiply(1, 2)); // 2
  console.log("1 / 2: ", divide(1, 2)); // 0 As the rust function is working with integers, we gonna receive 0 instead of 0.5

  return (
    <main>
      <h1>NextJs Rust Calc</h1>
    </main>
  );
}
```


现在代码是没有生效的，因为需要在一开始就初始化WASM,所以应该这样写


```tsx
"use client";
import { useEffect } from "react";
import init, { sum, subtract, multiply, divide } from "rust-calc";

export default function Home() {
  useEffect(() => {
    (async () => {
      await init();
      console.log("1 + 2: ", sum(1, 2)); // 3
      console.log("1 - 2: ", subtract(1, 2)); // -1
      console.log("1 * 2: ", multiply(1, 2)); // 2
      console.log("1 / 2: ", divide(1, 2)); // // 0 As the rust function is working with integers, we gonna receive 0 instead of 0.5
    })();
  }, []);

  return (
    <main>
      <h1>NextJs Rust Calc</h1>
    </main>
  );
}
```

为了使代码支持异步，从而支持更多场景，我们需要使用 ts 来打包 WASM, 在初始化之后导出所有函数

首先，创建一个 ts 包

```sh
  cd ../ && \
  mkdir ts-calc && \
  cd ts-calc && \
  npm init -y && \
  npm add typescript -D && \
  npx tsc --init && \
  touch index.ts
```

在 `tsconfig.json`, 设置 `"declaration": true`, 

在 `package.json`, 添加 `"types": "index.d.ts"`.

添加 rust 包依赖，

```sh
  npm add ../rust-calc/target/pkg-web
```


在`index.ts`中写入以下内容

```ts
  import * as rustCalc from "rust-calc";

  export const instantiate = async () => {
    const { default: init, initSync: _, ...lib } = rustCalc;

    await init();
    return lib;
  };

  export default instantiate;
```

编译ts工程

```sh
  npx tsc
```

在Next.js工程中直接移除 rust 包的依赖，然后添加 ts 打包的依赖


```sh
  cd ../nextjs-rust-calc && \
  npm remove rust-calc && \
  npm add ../ts-calc
```

`page.tsx`修改为以下内容

```tsx
  "use client";
  import { useEffect } from "react";
  import { instantiate } from "ts-calc";

  export default function Home() {
    useEffect(() => {
      (async () => {
        const { divide, multiply, subtract, sum } = await instantiate();
        console.log("1 + 2: ", sum(1, 2)); // 3
        console.log("1 - 2: ", subtract(1, 2)); // -1
        console.log("1 * 2: ", multiply(1, 2)); // 2
        console.log("1 / 2: ", divide(1, 2)); // 0 As the rust function is working with integers, we gonna receive 0 instead of 0.5
      })();
    }, []);

    return (
      <main>
        <h1>NextJs Rust Calc</h1>
      </main>
    );
  }
```

这确保了我们只能在初始化WASM方法后访问它。

这样就可以在各种环境下运行代码了。

参考:
  - [wasm-bindgen guide](https://rustwasm.github.io/wasm-bindgen/examples/hello-world.html)
  - [wasm-pack docs](https://rustwasm.github.io/docs/wasm-pack/)
  - [rustwasm guide](https://rustwasm.github.io/docs/book/)