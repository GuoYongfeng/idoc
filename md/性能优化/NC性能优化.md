# NC性能优化进度及问题详情汇总

## 实施进度


| # | 优化点 | 详细描述 | 优先级 | 跟进人 |  完成进度 |  备注说明 |
|---|---|---|---|---|---|---|
| 1 |js和css文件合并| 具体描述请浏览问题详情| 10 |传忠，王华|50%| |
| 2 |优化META信息| 具体描述请浏览问题详情 | 9 | 传忠 | 待确认 | |
| 3 |v_billform组件优化| 具体描述请浏览问题详情 | 9 |王华| 暂未完成 | |
| 4 |autocomplate的优化| 具体描述请浏览问题详情 | 9 |锐锋| 已完成 | |
| 5 | 优化大量事件绑定的问题 | 类似workflow.js 有15个click 事件这样的情形都需要修复 | 9 | chenylh | 待确认 | |
| 6 | 改变首屏资源的渲染方式 | 尝试由服务端渲染首屏页面，让用户能尽快看到页面的呈现，优化用户的首屏体验。| 10 | 传忠 | 暂未完成 | |
| 7 | 精简DOM结构 | 目前页面存在4000+个DOM元素，需要精简 | 9 |  | 未完成 | |
| 8 | 参照延迟加载 | 具体描述请浏览问题详情 | 9 | 待定 | 未完成 | |
| 9 | 资源延迟加载 |  延迟加载非首屏的静态资源 | 9 |  | 未完成 | |
| 10 | 设置Expires headers  | billform.controller.js文件设置的期限为**(2000/1/1) **、funcnodecard.js文件设置的期限为**(2000/1/1) **、 favicon.ico没有设置**(no expires) **| 9 | 待定 | 未完成 | |
| 11 | 文件未作压缩处理 |  还有10个文件未作压缩处理 | 9 |  | 未完成 | |
| 12 | 合理放置资源 | js资源放在页面底部 | 9 |  | 未完成 | |

## 问题详情

> **页面加载资源详情**

![资源描述](/img/nc_perform.png)

### 1. 文件合并

- 还有**26**个js文件
- 还有**9**个css文件


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


### 2. 优化META信息

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

### 3. 在服务器端使用v_billform组件而不是客户端

> 具体优化请结合业务功能考虑

### 4. autocomplate的优化

从DOM树上看，创建了大量的以下DOM元素

```html
<div class="ac_results" style="display: none; position: absolute; width: 180px;"></div>
```
### 5. 优化大量事件绑定的问题

目前单就workflow.js文件某个DOM上就发现绑定了有15个click 事件，需要着重排查类似的给DOM绑定大量事件的情况

### 6. 改变首屏资源的渲染方式

就付款单页面分析，由于使用requirejs将页面的主体部分异步的渲染出来，导致访问页面的首屏加载时间较长

> 建议尝试由服务端渲染首屏页面，让用户能尽快看到页面的呈现，优化用户的首屏体验。

### 7. 需要精简DOM结构

> 页面超过4000+的DOM元素

目前页面中存在大量的冗余DOM结构，需要合理布局，减少DOM数量

![](/img/dom.png)

### 8. 参照延迟加载

![](/img/modal.png)

### 9. 资源延迟加载

RequireJS 从 2.0 开始，也改成可以延迟执行

```
if (!'webkitFilter' in document.createElement('div').style ) {
    require.async('test',function(test){
      console.log(test.fuck);
    })
  }
```


### 10. 没有设置Expires headers

> Expires headers 告诉浏览器是否应该从服务器请求一个特定的文件或者是否应该从浏览器的缓存抓住它。Expires headers 的设计目的是希望使用缓存来减少HTTP requests的数量，从而减少HTTP相应的大小。

- http://localhost/iwebap/pages/20080EBR/billform.controller.js文件设置的期限为 **(2000/1/1) **
- http://localhost/.../funcnodecard.js文件设置的期限为 **(2000/1/1) **
- http://localhost/favicon.ico没有设置 **(no expires) **

### 11. 未作压缩处理

```
http://localhost/iwebap/css/workflow/workflow.css
http://localhost/uui2/libs/knockout/knockout-3.2.0.debug.js
http://localhost/iwebap/arap/js/urlutil.js
http://localhost/uui2/libs/i18next/i18next.js
http://localhost/iwebap/trd/bootstrap-table/src/bootstrap-table.js
http://localhost/iwebap/js/ref/jquery.scrollbar.js
http://localhost/iwebap/js/ref/refer.js
http://localhost/uui2/libs/uui/js/u.js
http://localhost/iwebap/js/sys/workflow/dateutil.js
```

### 12. 合理放置资源

> 问题描述：目前JS文件全部放在head标签内，应将js文件放在底部加载，css文件放在头部加载

![](/img/js.png)

### 13. BigPipe（暂不考虑）

> 分块加载的方案，需要结合业务场景使用。

### 14. 未使用cookie-free优化（暂不考虑）

工作原理说明：

当用户浏览器发送一个静态文件，如图片image、CSS样式表文件时会同时发送同一个域名（或二级域名）下的cookies，但是网站服务器对发送过来的cookies完全不予理会，`因此这些没用的cookies白白浪费了网站带宽，影响网站加载速度和网页性能表现。`

所以可以设置cookie-free domains来解决这个问题。`方法就是在浏览器发送静态内容的请求时不会发送cookies 的域名，一般会注册一个二级域名专门用来储存这些静态图片、JS、静态CSS文件。`

### 15. 没有使用CDN（暂不考虑）

静态资源44个，都没有走CDN
