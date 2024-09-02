# linux下如何使用UE5部署Cesium,并加载本地倾斜摄影数据

这里使用 Cesium World Terrain 和 Cesium OSM Buildings 构建 Cesium for Unreal 应用程序
1.Cesium for Unreal 插件安装到虚幻引擎
2.创建一个级别并从 Cesium ion 导入资产
3.设置项目的默认 ion 
4.使用 CesiumSunSky 为应用添加照明
5.使用 Cesium 的 DynamicPawn 导航关卡
## step.1 先决条件

Ue版本高于5.1版本，有部署高版本的cmake,一个Cesium ion账户。
其他平台使用者可以直接通过虚幻引擎市场免费下载Cesium for Unreal插件与Cesium for Unreal Samples项目
Linux平台则需要到官网下载插件，并于Unreal启动项目中Plugin文件夹新建一个Marketplace文件夹，将
CesiumForUnreal存入其中(如果已有则直接存入，无需新建)，最终路径类似于 UE_5.3/Engine/Plugins/
Marketplace/CesiumForUnreal.

接下来在虚幻引擎中创建游戏作为类别的项目，可以搜素到Cesium for Unreal插件，如果未搜索到，则是存入路径
存在问题，需要对上述操作进行检查。

激活插件并重启编辑器后，可能会出现与水体碰撞配置文件相关的错误。单击将条目添加到 DefaultEngine.ini链接
以修复它。

而后创建新的空关卡,禁用世界边界检查。此选项位于世界设置（窗口 -> 世界设置）中的世界 -> 高级类别中。
虚幻引擎将尝试让远离原点的 Pawn 飞回原点。在大多数 Cesium 应用程序中，远离原点是完全正常的，这种自动行
为会阻止 Pawn 到达用户希望它去的地方。

通过点击Windows可以打开Cesium面板，这里需要使用到需要准备的Cesium ion账户，点击Connect to Cesium ion进
行登陆，就可以在Cesium的商场中进行下载并在Cesium中使用，包含有澳大利亚倾斜摄影，高密度楼宇，日本倾斜摄
影数据等。将为项目创建一个默认访问令牌。从 Cesium ion 流式传输的每个资产都需要一个访问令牌。这里将设置
一个项目范围的访问令牌，所有资产都将使用该令牌。单击Cesium面板顶部的令牌按钮。将出现一个新窗口来配置令
牌。选择“创建新令牌”选项，并根据需要重命名令牌。然后，按创建新项目默认令牌按钮。

需要添加一些灯光，以便能够在后续步骤中看到添加的图块集。Cesium for Unreal 附带一个预制的、可感知地球的
太阳和大气系统，称为CesiumSunSky。在面板的“快速添加基本演员”部分中，找到Cesium SunSky并单击按钮将其添加
到关卡中。应该会看到视口中出现类似天空的渐变。CesiumSunSky 使用真实的光强度值，比标准虚幻项目亮得多。因
此，在某些项目设置下，光线会使场景褪色并使其呈现白色。要解决此问题，请在自动曝光设置选项中启用自动曝光
和扩展默认亮度范围。或者可以通过转到编辑 -> 项目设置，然后导航到引擎 -> 渲染并向下滚动到默认设置部分来
找到这些选项。或者，可以将太阳光的强度降低到更合理但不太现实的值。单击CesiumSunSky。在“详细信息”面板
中，找到“定向光”组件，并将光的强度降低到 10.0。

单击编辑器左上角的将当前关卡保存到磁盘按钮，或按 Ctrl+S。为关卡命名。如果想将新关卡设置为项目的默认
地图。这将确保您的关卡在您重新启动项目或打包并运行应用程序时自动打开。在编辑 -> 项目设置窗口左侧边栏
中，单击地图和模式，然后查找默认地图部分。将编辑器启动地图和游戏默认地图都更改为您的新关卡。

## step.2 添加Cesium资产

使用Cesium面板，通过单击该条目旁边的按钮添加“Cesium World Terrain + Bing Maps 航空图像”。查看右侧的大纲
视图。除了之前添加的CesiumSunSky外，还能看到各种 Cesium 角色。其中之一Cesium World Terrain是刚刚创建的
图块集。其他三个（CesiumCameraManager、CesiumCreditSystemBP和CesiumGeoreference）是在您第一次向关卡添
加 3D 图块集或地理参考角色时自动创建的。这是一个Cesium3DTileset演员。它将 3D Tiles 数据流式传输到虚幻引
擎中，并提供一种配置该图块集的方法。

1.在大纲视图中选择CesiumGeoreference角色。该角色决定了关卡设置在世界的哪个位置。可以使用该角色更改关卡
当前的纬度、经度和高度。
2.在详细信息面板中，查找Cesium 类别下的原点纬度、原点经度和原点高度变量。可以改为Origin Latitude = 39.
9236,Origin Longitude为116.4074,Origin Height 为500，可以来到北京上空500m.

3.城市看起来很平坦。Cesium World Terrain 不包含建筑细节。可以使用Cesium OSM Buildings，从Cesium Quick 
Add面板中，将Cesium OSM Buildings添加到关卡中。这时其他平台可以看到城市白模，与建筑相关，而linux下则会
出现空指针的报错,而无法生成白模，这里推荐使用google的倾斜摄影数据，但是没有国内的数据，所以需要使用国内
数据如天地图,水经注，或者本地数据添加。

4.要移动视口的相机，请在视口窗口中单击并按住鼠标右键。按住鼠标右键的同时，可以通过移动鼠标环顾四周，并
通过按键盘上的 W、A、S 和 D 键来飞行。可以使用鼠标滚轮修改相机速度。可以使用视口右上角的相机速度设置以
更大的增量修改相机速度

5.如果按下“播放”按钮，默认相机速度非常慢，并且无法调整。考虑到现实世界数据的规模，您需要一个不同的Pawn
才能在游戏过程中有效地导航关卡。
使用Cesium面板，添加一个动态 Pawn。DynamicPawn是一个地理参考Actor。它保持相对于地球坐标而不是标准虚幻引
擎世界坐标的位置。这意味着如果将地理参考原点更改为其他位置，DynamicPawn将保持不变。如果想将其移动到新位
置，请在大纲视图中选择它并将其位置 X、Y和Z坐标设置为 0，或使用灰色箭头将其位置重置为原点。

## step.3 设定位置与跳转

Ue5中存在Geoference组件，模型与恐龙头类似，可以将Geoference组件的位置与初始位置尽可能贴近，并将数据与
Origin latitue、Origin Longitude、Origin Height一致，则可以通过Geoference组件来查看当前经纬度。e这时x为
东，-y为北。

可设置Geoference控件蓝图与关卡蓝图。Cesium插件带有飞的操作，可以设置位置后，加入点击事件，来进行位置跳
转。这里可以在控件蓝图中制作按钮，并生成点击事件，于a关卡蓝图中控制。并且可以在控件蓝图中进行数据处理,
可以自行设置原点，将Cesuim飞行的地理位置设置改为获取Geoference控件文本框输入的数据，并可以设定多处按钮
点击事件，包括不同位置设定飞行，或是固定位置飞行。可以进行转换，得到与原点位置差，与当前位置经纬度，进
行原点数据处理。

## step.3 自行数据设定。

以天地图为例，注册账户后会得到1天1w次限制的各个地理模型访问次数。但是由于ue是连续进行访问的，1w次只能h
支持10几分钟。并且由于天地图加有限制，可以访问模型api获取到user-agent的header,于本地数据处理成新的api访
问，就可以访问了.
本地离线数据：下载dem数据，用cesiumlab切片，用file:///做前缀加载layer.json，把加载方式改成from url