<h1 style="font-size: 40px;text-align:center;color: #007cdc;">React 学习指南</h1>

<img src="/img/react/react.png" />


**React 系列课程**面向初中级前端开发人员以及感兴趣的开发者，我们希望你具备 HTML, CSS 和 JavaScript 的Web 开发的基础，同时希望你具有以下的基本开发环境配置以及基本知识储备。如果对课程及内容有任何反馈，请到[这里](https://github.com/GuoYongfeng/idoc/issues/3)。

接下来，我们将从语言特性、代码风格、构建工具、依赖管理、路由管理、核心类库、状态管理、CSS预处理、API 工具库、测试工具等前端开发的方方面面进行技术梳理，为你挑出这些最佳实践并规划面向未来的技术学习之路。但同时，面对剧烈变革同时又日趋稳定的前端新技术，有两句话和大家共享：1. 对于新技术，请确保你需要，再去使用；2. 保持简单，每次学一样，深入理解和使用。

## 语言：ES6 + Babel编译器

ECMAScript 6 是 JavaScript 的最新版本。由于 ES6 很新，你可能还没学习到，浏览器也可能尚未兼容，但别担心，通过Bebel编译器和你的打包工具结合，将能够为你自动转换成兼容代码。

Babel是个可插拔ES6编译器，配置合适的预设（babel-preset-es2015，babel-preset-react....），我们就可以开动了。

## 代码风格：ESlint

使用ESlint配合Airbnb指南来保持我们团队的代码风格。

我强烈建议你使用 Airbnb的风格指南，大部分可以被 ESLint airbnb config 来严格约束实现。如果你们团队会在代码风格上产生分歧和争议，那么拿出这份指南来终结所有的不服！它也不是完美的（因为完美的风格不存在），但高度推荐这个规范来保持团队一致的代码风格。

一旦你开始熟悉它，我建议你开启更多的规则。在编辑撰写代码时候越多的捕获不规范（配置你的编辑器IDE使用上这个ESLint插件），就会避免分歧和在决定费神，从而让你和团队更加高效！

## 依赖管理：NPM

这一点很明确，就用NPM，因为这意味着我们可以充分的使用NPM上20w+的优秀第三方package。

类似与 Browserify 和 Webpack 的构建工具把npm的强大功能引到了web上。版本处理变得很简单，你也获得了node生态的大部分模块提供的功能。不过CSS的相关处理还是不够完美。

你可能会考虑的一件事：如何在开发服务器构建应用。npm 常使用通配符来指定版本号，导致你开始书写代码到最后部署时候版本号很可能已经变化了。使用 shrinkwrap 文件来锁定你的依赖（我建议使用 [Uber的 shrinkwrap](https://github.com/uber/npm-shrinkwrap) 来获得更一致性的输入）。

同时考虑使用利用类似于 [Sinopia](https://www.npmjs.com/package/sinopia) 来构建自己的私有npm服务器。Babel可以把 ES6 模块语法编译到CommonJS。意味着你可面向未来的语法，和在使用构建工具（如Webpack 2.0）时获得它支持的一些静态代码分析工具如 [tree shaking](http://www.2ality.com/2015/12/webpack-tree-shaking.html) 的优势

## 构建工具：Webpack

不想在你的页面文件中加入非常多的外链Script引用，那你就需要一个构建工具来打包你的依赖。如果你也需要允许npm包在浏览器运行工作的工具，那么Webpack就是你需要的。

一年前，关于这块还有不少选择。根据你的开发环境，你可以利用Rails的sprockets来解决类似问题。RequireJS，Browserify和Webpack都是以JavaScript基础的工具，现在RollupJS声称自己在ES6模块上处理的更好。

在一个个尝试使用后，我最终还是高度推荐Webpack：
- 它有自己独特的写法，不过即使是最少见的场景也能找到解决的方法
- 主流的模块格式都支持（如AMD，CommonJS，Globals写法）
- 它有处理有问题模块的能力（不是标准的模块写法
- 它能处理CSS文件
- 它有最完善的cache busting/hashing 系统（如果你需要把你的资源发布到CDN）
- 它内置热加载功能
- 它能加载几乎你需要的一切
- 它一套令人惊叹的优化列表

Webpack目前也是处理大型SPA应用项目的最好方案，利用它的代码切割（code splitting）和懒加载特性。

`不过它的学习曲线很陡峭，但是一点你掌握了，你还是会认为非常值得因为它的强大功能。`

> 那么Gulp或Grunt呢？ Webpack比起来最适合处理静态资源。所以他们开始可以用来跑一些其他的任务（但是也不推荐），现在更简单的方法是直接用上 [npm scripts](https://docs.npmjs.com/cli/run-script)


## 核心类库：React


React在目前前端技术所表现出的亮点吸引了大批的开发者：
- 从顶到底都是组件，你的应用程序代码非常容易理解
- 学习曲线非常平缓，要知道罗列它所有关键的API都不会超过一页A4纸张。
- JSX非常棒，你可以用获得JavaScript编程语言的能力和工具链来描述页面
- 基于React的项目实现，除了组件化、虚拟DOM等特点在代码复用以及性能上带来的一般好处外，React的思想使得开发者之间可以更好的分工与合作，在配合上非常顺畅。
- 规范的接口以及极强的约束，使得整个代码结构清晰，不同开发者的代码高度一致。
- 技术生态。越来越多的开发调试工具以及JavaScript框架正在依附着react生长，这一技术选型对以后的产品升级以及客户端的开发上线打下了良好的基础。
- 这套技术实现，框架库代码压缩后大于200K,gzip后实际传输大小为60K+，更适合大型的webapp。
- 配合ES6的使用，可以大幅提高编程效率，降低很多之前的代码坑。
- 有很优秀的开源组件库可以使用：`Antd Materialui React-bootstrap Amaze-ui Sematic-ui`

现在不少全能的大型框架如Ember，Angular，它们承诺说帮你处理所有的事。但是在React生态中，尽管需要对组件做一些决定，但是这方案更强壮。同时，很多其他框架，譬如Angular2.0等，正在快速追赶React。

而且，我想强调的是：**谁能做到真正的跨平台，谁就能笼络更多的人心**。

有了React的实践之后，你可以继续使用React Native来开发应用。我想，你的老板会考虑给你加薪的，因为你的能力对公司越来越重要。

## 路由管理：React-router

“单页面应用” 是时下的技术热点. 当网页加载完成, 用户点击链接或者按钮的时候, JavaScript 会更新页面和改变地址栏, 但网页不会刷新. 地址栏的管理就是通过 **路由(router)** 来完成的.

目前 React 生态圈最受欢迎的路由解决方案是 [react-router](https://github.com/rackt/react-router)，不过如果你也在用Redux来构建你的应用，也强烈建议你关注react-router-redux和redux-simple-router，它们会帮你解决一些头疼的问题。

**如果你创建的并非单页面应用, 请不要使用路由.** 无论如何, 大部分项目都是从大型应用中的小组件开始的.

## 应用生命周期：Redux

现在我们有了我们的视图和组件层，应用程序还需要管理数据状态和应用的生命周期。Redux也是毋容置疑的优胜者。除了React，Facebook展示了名叫Flux的单向数据流的设计模式。Flux最早用来解决和简化应用的状态管理，但是随之而来，很多开发者提出了不少新的问题如如何存储数据状态和从哪发送Ajax请求。

为了解决这些问题，不少基于Flux模式之上的框架诞生了：Fluxible, Reflux, Alt, Flummox, Lux, Nuclear, Fluxxor 还有很多。

不过，这其中的类Flux的优雅实现最终赢得了社区的关注，它就是Redux。最重要的是，学习Redux小菜一碟。它的作者，Dan Abranmov是一位非常棒的老师（他的教学视频非常易懂好学）。通过那些视频你很容易成为Redux的专家。

我见识到一组几乎没有任何React开发经验的工程师通过学习他的视频，在几周内就开发好能上线的React项目（代码质量也是顶级的）Redux的生态系统和Redux本身一样优秀。从神乎其神的开发者工具到令人记忆深刻的reselect，Redux的社区是你最好的后盾。

不过有一点需要注意的是不要轻易的去尝试抽象Redux的项目模板。那些模板背后都是有意义有原因的。所以你尝试盲目修改前确保你已经使用过它和理解这样组织代码背后的原因。

## 测试：Mocha + Chai + Sinon

目前在 JavaScript 单元测试上，我们有众多选择，你选择任何一个都不会错太多。因为你开始做单元测试，你就走对一大步了。

我们可以有这些选择：Jasmine，Mocha，[Tape](https://github.com/substack/tape)，[AVA](https://github.com/sindresorhus/ava) 和 Jest。

我对一个测试框架的的选择标准是：
- 它应该可以在浏览器中运行从而方便调试
- 它应该运行的足够快- 它可以很好的处理解决异步测试
- 它可以在命令行就能很方便的使用
- 它应该允许我使用任意断言库，而不会约束限制

我倾向于 Chai 断言因为它拥有很多插件，Mocha的异步测试支持很棒（你不用在写类似于 done callback之类的）。

[Chai as Promised](https://github.com/domenic/chai-as-promised) 也是很屌。我强烈建议你使用 [Dirty Chai](https://github.com/prodatakey/dirty-chai) 来避免一些让人头疼的问题。

Webpack的 [mocha loader](https://github.com/webpack/mocha-loader) 让你编码时自动执行测试。对于React而言，Airbnb的[Enzyme](https://github.com/airbnb/enzyme)和Teaspoon（不是Rails那个！）是不错的相关工具选择。

我非常喜欢Mocha的特性和支持情况。如果你喜欢一些最小主义的，读读这篇[关于tape的文章](https://medium.com/javascript-scene/why-i-use-tape-instead-of-mocha-so-should-you-6aa105d8eaf4)

## 工具库：Lodash

JavaScript不像Java或.NET上有很多强大的内置工具集。所以你可能需要引入一个。

Lodash，目前来说应该是杂七杂八都有的首选。同时它类似注入[懒求值](http://filimanjaro.com/blog/2014/introducing-lazy-evaluation/)这样的特性让它成为性能最高的选择之一。如果你不想用你就不要把它全部都导入，同时：lodash能让你仅仅引入哪些你需要的函数（这点尤其重要考虑到它现在变得越来越大）。

随着4.x版本带来，它原生支持可选的函数式模式给那些函数式编程的极客们使用。来看看[怎么使用它](https://github.com/lodash/lodash/wiki/FP-Guide)，同时，[lodash 中文文档](http://lodashjs.com/docs/)应该能够让你很快的上手。

如果你真的很喜欢函数式编程，那么不管怎么样，留意下优秀的[Ramda](http://ramdajs.com/0.19.1/index.html)。

## HTTP请求：fetch

许多React应用再也不需要jQuery了。除非你需要使用一些遗留的老旧的第三方组件（它依赖于jQuery），因为根本没必要。同时，意味着你需要一个类似于$.ajax的替代品。

为了保持简单，仅仅使用[fetch](https://fetch.spec.whatwg.org/)，它在Firefox和Chrome中内建支持。对于其他浏览器，你可能需要引入polyfill。

我推荐使用[isomorphic-fetch](https://github.com/matthew-andrews/isomorphic-fetch) 来在服务器端在内覆盖了基础组件选择。还有一些其他好的类库选择如[Axios](https://github.com/mzabriskie/axios)，但目前在fetch之上没有多余需求。

## 样式：考虑CSS模块

在 React 出现之前, 很多人通过像 SASS 这样的预处理器来重用复杂的 CSS 样式表. 鉴于 React 使开发可重用组件变得容易, 你的样式表可以变得没那么复杂了. 社区中许多人正尝试完全抛弃样式表.

*当你开始用 React 的时候, 只要用你平常使用的方法去写就好了.*React 就像其他 JavaScript 库一样, 可以和 CSS 预处理器很好地配合工作.

有一点需要特殊注意的是，CSS Modules。它限制了CSS的层叠部分，使得我们可以定义更加明确的依赖，来避免冲突。你再也不用担心Class名称一致导致的覆盖，也不用特意为了避免它而添加额外的前缀。它和React也配合的很好。有个不足：css-loader和css modules一起使用会导致非常缓慢，如果你的样式数量不少，那么在它优化之前还是避免使用它吧。

## 数据结构：Immutable.js

[Immutable.js](https://facebook.github.io/immutable-js/) 提供了一系列的数据结构, 以帮助解决构造 React 应用时的某些性能问题. 这是一个很棒的库, 你可能会在应用发展的过程里大量用到它, 但直到你在意识到性能问题以前, 它是完全不必要的.

## 前后端同构

Universal或Isometric的JavaScript代表着JavaScript写的代码可以被同时运行在客户端和服务器上。这需要用来在服务器端预先渲染页面来提升性能或SEO友好。

感谢React，之前只有类似于Ebay或Facebook这样的巨型科技公司才能实施的方案，现在很多小的开发团队都能做到。不过它并不那么容易，它显著的加大了复杂性，限制了你类库和工具选择。如果你在构建B2C的网站，类似于电商网站，那么你可能必须使用这个方案。但如果你是在做内部网站或者是B2B应用站点，那么这样的性能提升和SEO其实并不需要。所以和你的项目经理讨论下，看看是否有必要~

## 参考资料

1.《[展望 Javascript 2016年的趋势和生态发展](https://github.com/gaohailang/blog/issues/12)》，gaohailang同学译著，英文原文为《[State of the Art JavaScript in 2016](https://medium.com/javascript-and-opinions/state-of-the-art-javascript-in-2016-ab67fc68eb0b)》

2.《[如何学习React](https://github.com/petehunt/react-howto/blob/master/README-zh.md)》，Zhangjd同学译著，原文为《[react-howto](https://github.com/petehunt/react-howto)》
