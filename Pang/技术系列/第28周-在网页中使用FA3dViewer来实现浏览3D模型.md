# ES5中的使用
适用于原生Javascript下的前端项目开发

## step.1 安装或引用核心库
```html
<!-- 本库的使用，先引入基础库及基础样式 -->
<link href="https://www.forface3d.com/forface-libs/forface3dViewer/last/style.css" rel="stylesheet" />
<script src="https://www.forface3d.com/forface-libs/forface3dViewer/last/viewer3D.min.js"></script>
```
## step.2 开发代码示例
```html
<div id="forface-container-div" style="height: 100%"}></div>
<script>
    // 初始化
    Forface3dViewer.Init('forface-container-div', {
        // 必须的
        appId: `<You AppID>`,
        secretKey: `<You SecretKey>`
    }).then(function(){
      // 加载模型
      Forface3dViewer.viewer.loadModel(`<ModelKey>`)
    })
</script>
```