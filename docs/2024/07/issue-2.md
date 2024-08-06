# 一个轻量、可拓展、针对手机网页的前端开发者调试面板-vConsole

vConsole 是微信小程序的官方调试工具
特性：

- 日志(Logs)： console.log|info|error|...
- 网络(Network)： XMLHttpRequest, Fetch, sendBeacon
- 节点(Element)： HTML 节点树
- 存储(Storage)： Cookies, LocalStorage, SessionStorage
- 手动执行 JS 命令行
- 自定义插件

## 安装（推荐）

```
    npm install vconsole
```

## 使用

```
//angular json引用
"scripts": ["node_modules/vconsole/dist/vconsole.min.js"]

import VConsole from 'vconsole';

ngOnInit() {
  if(window.VConsole){ // 显示VConsole
      window.VConsole = new VConsole();
      let vcButton = document.getElementById('__vconsole');
      vcButton.className = "";
  }
  //或者
  const vConsole = new VConsole({ theme: 'light' });
  console.log('Hello world');
  //可以销毁
  vConsole.destroy();
}


```

其余调试就可以像 web 一样了，可以查看 console.log，查看网络请求，查看 cookie，查看 localStorage 等。

## 参考

https://github.com/Tencent/vConsole

# 一个开源的 UI 网站（Open-Source UI elements for any project）

有很多 ui 设计和动效（https://uiverse.io/）
