# arkTS 入门

## 下载与安装

**官方下载地址：[https://developer.huawei.com/consumer/cn/deveco-studio/](https://developer.huawei.com/consumer/cn/deveco-studio/)**
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/44ea16628707426c8af53f8fb575bf98.png)
下载完成过后进行安装，安装很简单不在此复述。大致就是一直下一步，需要注意的就是需要node环境，没有的话也可以等开发工具给你安装。
## 使用
## 汉化
在设置>插件>已安装里面找到简体中文，默认是没有开启的，你点开就可以了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/37a1cb8de88347b4840c10be3927cabd.png)
## 开始使用
### 项目结构（Stage模型）
可以去官方文档进行查看。
[https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V2/start-overview-0000001478061421-V2](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V2/start-overview-0000001478061421-V2)![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/6249037c8d06406799df043ec21ace8a.png)
- AppScope > app.json5：应用的全局配置信息。
- entry：HarmonyOS工程模块，编译构建生成一个HAP包。
- src > main > ets：用于存放ArkTS源码。
- src > main > ets > entryability：应用/服务的入口。
- src > main > ets > pages：应用/服务包含的页面。
- src > main > resources：用于存放应用/服务所用到的资源文件，如图形、多媒体、字符串、布局文件等。关于资源文件，详见资源分类与访问。
- src > main > module.json5：Stage模型模块配置文件。主要包含HAP包的配置信息、应用/服务在具体设备上的配置信息以及应用/服务的全局配置信息。具体的配置文件说明，详见module.json5配置文件。
- build-profile.json5：当前的模块信息、编译信息配置项，包括buildOption、targets配置等。其中targets中可配置当前运行环境，默认为HarmonyOS。
- hvigorfile.ts：模块级编译构建任务脚本，开发者可以自定义相关任务和代码实现。
- oh_modules：用于存放三方库依赖信息。关于原npm工程适配ohpm操作，请参考历史工程迁移。
- build-profile.json5：应用级配置信息，包括签名、产品配置等。
- hvigorfile.ts：应用级编译构建任务脚本。
## 真机运行
查看官方文档。最好去成为开发者，用华为账户登录会自动生产签名就不需要那么麻烦了。
**[https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V2/start-with-ets-stage-0000001477980905-V2#section588321874518](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V2/start-with-ets-stage-0000001477980905-V2#section588321874518)**

# 基本语法概述
![https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20240718193931.10297359816076517902093055040456:50001231000000:2800:DE9CEFDA40283100B22019F065E16B6D777F6B36051444CAD8FA74852F8F0A1F.png?needInitFileName=true?needInitFileName=true](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20240718193931.10297359816076517902093055040456:50001231000000:2800:DE9CEFDA40283100B22019F065E16B6D777F6B36051444CAD8FA74852F8F0A1F.png?needInitFileName=true?needInitFileName=true)
- 装饰器： 用于装饰类、结构、方法以及变量，并赋予其特殊的含义。如上述示例中@Entry、@Component和@State都是装饰器，@Component表示自定义组件，@Entry表示该自定义组件为入口组件，@State表示组件中的状态变量，状态变量变化会触发UI刷新。

- UI描述：以声明式的方式来描述UI的结构，例如build()方法中的代码块。

- 自定义组件：可复用的UI单元，可组合其他组件，如上述被@Component装饰的struct Hello。

- 系统组件：ArkUI框架中默认内置的基础和容器组件，可直接被开发者调用，比如示例中的Column、Text、Divider、Button。

- 属性方法：组件可以通过链式调用配置多项属性，如fontSize()、width()、height()、backgroundColor()等。

- 事件方法：组件可以通过链式调用设置多个事件的响应逻辑，如跟随在Button后面的onClick()。

- 系统组件、属性方法、事件方法具体使用可参考基于ArkTS的声明式开发范式。

## 声明式UI
 和安卓有一点相似。链式写法，如：
 ```
 Image('test.jpg')
  .alt('error.jpg')    
  .width(100)    
  .height(100)
 ```
如果有用到的自定义组件就如下：
```
@Component
struct HelloComponent {
  @State message: string = 'Hello, World!';

  build() {
    // HelloComponent自定义组件组合系统组件Row和Text
    Row() {
      Text(this.message)
        .onClick(() => {
          // 状态变量message的改变驱动UI刷新，UI从'Hello, World!'刷新为'Hello, ArkUI!'
          this.message = 'Hello, ArkUI!';
        })
    }
  }
}
注意：HelloComponent可以在其他自定义组件中的build()函数中多次创建，实现自定义组件的重用。如下：

class HelloComponentParam {
  message: string = ""
}

@Entry
@Component
struct ParentComponent {
  param: HelloComponentParam = {
    message: 'Hello, World!'
  }

  build() {
    Column() {
      Text('ArkUI message')
      HelloComponent(this.param);
      Divider()
      HelloComponent(this.param);
    }
  }
}
```
装饰器例如
- @Builder装饰器：自定义构建函数
- @BuilderParam装饰器：引用@Builder函数
- @Styles装饰器：定义组件重用样式
- @Extend装饰器：定义扩展组件样式
- @AnimatableExtend装饰器：定义可动画属性
- @Require装饰器：校验构造传参

用法：https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-builder-V5