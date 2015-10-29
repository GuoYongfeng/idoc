# 性能优化建议及进度汇总

## 文件合并

|跟进人|完成进度|
|---|---|
|李传忠，郭永峰，王华|暂未完成|

> 方案描述：

- 技术平台：合并以下js文件
```javascript
http://20.1.85.11:8088/uui2/libs/tangram/event.js
http://20.1.85.11:8088/uui2/libs/tangram/loader.js
text.js css.js
```

- 应用平台：合并以下js文件
```
http://20.1.85.11:8088/iwebap/js/templetutils.js
http://20.1.85.11:8088/iwebap/js/billmanagemodel.js
http://20.1.85.11:8088/iwebap/js/BillFormActionUtils.js
http://20.1.85.11:8088/iwebap/js/EventConst.js
http://20.1.85.11:8088/iwebap/js/sys/workflow/dateutil.js
http://20.1.85.11:8088/iwebap/js/sys/workflow/workflow.js
http://20.1.85.11:8088/iwebap/js/ref/ncReferComp.js
http://20.1.85.11:8088/iwebap/js/ref/refDList.js?_=1445945847436
http://20.1.85.11:8088/iwebap/js/sys/formula.js
http://20.1.85.11:8088/iwebap/js/ext/gridRender.js
http://20.1.85.11:8088/iwebap/js/billform.js
http://20.1.85.11:8088/iwebap/js/sys/webcontext.js
http://20.1.85.11:8088/iwebap/js/ext/fullgridFormatter.js
http://20.1.85.11:8088/iwebap/js/ext/billutil.js
http://20.1.85.11:8088/iwebap/js/ext/fullgridRender.js
```

- 业务相关人员：合并以下js和css文件

```
// JS部分
http://20.1.85.11:8088/iwebap/arap/js/commonctrl.js
http://20.1.85.11:8088/iwebap/arap/js/urlutil.js

// css部分
http://20.1.85.11:8088/iwebap/css/style.css
http://20.1.85.11:8088/iwebap/css/ref/ref.css
http://20.1.85.11:8088/iwebap/page/fi/temp.css
http://20.1.85.11:8088/iwebap/css/template/billtemplateform.css
http://20.1.85.11:8088/iwebap/css/workflow/workflow.css
```

## JS文件放在底部加载

> 问题描述：目前JS文件全部放在head标签内

![](/img/js.png)

## 优化META信息

|跟进人|完成进度|
|---|---|
|郭永峰，李传忠|已整理优化点|

> 方案描述：

```
<meta http-equiv="Content-Type" content="text/html;charset=utf-8">
<!-- 避免IE使用兼容模式 -->
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
<!-- 启用360浏览器的极速模式(webkit) -->
<meta name="renderer" content="webkit">
<!-- viewport设置 -->
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="keywords" content="关键词">
<meta name="description" content="描述内容">
<!-- ios在网页加载时隐藏地址栏与导航栏，如有ipad展示可考虑加上 -->
<meta name="viewport" content="minimal-ui">
<!-- 以下几个为移动端的meta优化，结合业务考虑使用 -->
<meta content="telephone=no" name="format-detection">
<meta content="address=no" name="format-detection">
<meta name="apple-mobile-web-app-status-bar-style" content="black">
<meta name="apple-mobile-web-app-capable" content="yes">
<!-- 是否启动webapp功能，会删除默认的苹果工具栏和菜单栏。 -->
<meta name="apple-mobile-web-app-capable" content="yes" />
<!-- 显示手机信号、时间、电池的顶部导航栏的颜色。默认值为default（白色），可以定为black（黑色）和black-translucent（灰色半透明） -->
<meta name="apple-mobile-web-app-status-bar-style" content="black" />
<!-- 忽略页面中的数字识别为电话号码或是邮箱 -->
<meta name="format-detection" content="telphone=no, email=no" />
<!-- 针对手持设备优化，主要是针对一些老的不识别viewport的浏览器，比如黑莓 -->
<meta name="HandheldFriendly" content="true">
<!-- 微软的老式浏览器 -->
<meta name="MobileOptimized" content="320">

```

## 在服务器端使用v_billform组件而不是客户端

|跟进人|完成进度|
|---|---|
|王华|暂未完成|

## autocomplate的优化

|跟进人|完成进度|
|---|---|
|丁锐锋|暂未完成|

从DOM树上看，创建了大量的以下DOM元素

```html
<div class="ac_results" style="display: none; position: absolute; width: 180px;"></div>
```
## workflow.js 有15个click 事件

|跟进人|完成进度|
|---|---|
|chenylh|暂未完成|

> 需要排查类似的给DOM绑定大量事件的情况

## 改变首屏资源的渲染方式

|跟进人|完成进度|
|---|---|
|传忠|暂未完成|

就付款单页面分析，由于使用requirejs将页面的主体部分异步的渲染出来，导致访问页面的首屏加载时间较长

> 建议尝试由服务端渲染首屏页面，让用户能尽快看到页面的呈现，优化用户的首屏体验。

## 精简DOM结构

目前页面中存在大量的冗余DOM结构，需要合理布局，减少DOM数量

![](/img/dom.png)

## 信息弹框延迟加载

![](/img/modal.png)
