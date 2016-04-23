
<h1 style="font-size: 40px;text-align:center;color: #007cdc;"> 基于React + Redux的全栈开发 </h1>


> 声明：本文内容基于kazaff翻译的博文[Full-Stack Redux Tutorial](http://teropa.info/blog/2015/09/10/full-stack-redux-tutorial.html)修改，内容仅供交流学习。

这是一次充实而又充满挑战的旅程，我们将要一起创建一个全栈的`Redux`和`Immutable.js`应用。我们将详细讲解创建该应用的`Node + Redux`后端和`React + Redux`前端的所有步骤。

在项目的开发过程中，我们将使用`ES6、Babel、Socket.io、Webpack、Mocha和Redux`等技术。这是一个非常令人着迷的技术栈选型，你肯定不及待的想要开始了。

## 1.你需要准备什么

你需要安装Node和NPM，并且下载一款你喜欢的编辑器。

- [x] [环境安装](/idoc/html/团队/新员工入职指南.html)
- [x] [Nodejs环境配置和Git基本配置](/idoc/html/技术分享/Nodejs环境配置和Git基本配置.html)

这篇文章需要读者具备开发js应用的能力，我们将使用如下技术：

- [x] [Node](/idoc/html/技术分享/Node.js知识详解.html)
- [x] [ES6](/idoc/html/React课程专题/快速搞定ES6基本语法.html)
- [x] [React](/idoc/html/React课程专题/React基础知识详解.html)
- [x] [Babel](/idoc/html/React课程专题/Babel使用指南.html)
- [x] [Webpack](/idoc/html/React课程专题/使用Webpack搭建前端开发工作流.html)
- [x] [Redux](/idoc/html/React课程专题/让Redux来管理你的应用（一）.html)

所以你最好能了解这些工具，这样你才不会掉队，跟上大家的节奏。

## 2.应用功能分析

我们将要开发一款应用，它用来为聚会，会议，集会等用户群提供实时投票功能。

这个点子来自于现实中我们经常需要为电影，音乐，编程语言等进行投票。该应用将所有选项两两分组，这样用户可以根据喜好进行二选一，最终拿到最佳结果。

举个例子，这里拿Danny Boyle电影做例子来发起投票：

![](http://teropa.info/images/vote_logic.png)

这个应用有两类独立的界面：用于投票的移动端界面，用于其它功能的浏览器界面。投票结果界面设计成有利于幻灯片或其它更大尺寸的屏幕显示，它用来展示投票的实时结果。

![](http://teropa.info/images/vote_system.png)

## 3.技术基础架构

该系统应该有2部分组成：

- 浏览器端我们使用React来提供用户界面
- 服务端我们使用Node来处理投票逻辑。

两端通信我们选择使用WebSockets。

我们将使用Redux来组织前后端的应用代码。
我们将使用Immutable数据结构来处理应用的state。

虽然我们的前后端存在许多相似性，例如都使用Redux。但是它们之间并没有什么可复用代码。这更像一个分布式系统，靠传递消息进行通信。

## 4.开发服务端应用

我们先来实现Node应用，这有助于我们专注于核心业务逻辑，而不是过早的被界面干扰。

实现服务端应用，我们需要先了解Redux和Immutable，并且明白它们如何协作。Redux常常被用在React开发中，但它并不限制于此。我们这里就要学习让Redux如何在其它场景下使用。

我推荐大家跟着我们的指导一起写出一个应用，但你也可以直接从[github](https://github.com/teropa/redux-voting-server)上下载代码。

### 4.1 设计应用的状态树（State Tree）

设计一个Redux应用往往从思考应用的状态树数据结构开始，它是用来描述你的应用在任何时间点下状态的数据结构。

任何的框架和架构都包含状态。在Ember和Backbone框架里，状态就是模型（Models）。在Anglar中，状态常常用Factories和Services来管理。而在大多数Flux实现中，常常用Stores来负责状态。那Redux又和它们有哪些不同之处呢？

最大的不同之处是，在Redux中，应用的状态是全部存在一个单一的树结构中的。换句话说，应用的所有状态信息都存储在这个包含map和array的数据结构中。

这么做很有意义，我们马上就会感受到。最重要的一点是，这么做迫使你将应用的行为和状态隔离开来。状态就是纯数据，它不包含任何方法或函数。

这么做听起来存在局限，特别是你刚刚从面向对象思想背景下转到Redux。但这确实是一种解放，因为这么做将使你专注于数据自身。如果你花一些时间来设计你的应用状态，其它环节将水到渠成。

这并不是说你总应该一上来就设计你的实体状态树然后再做其它部分。通常你最终会同时考虑应用的所有方面。然而，我发现当你想到一个点子时，在写代码前先思考在不同解决方案下状态树的结构会非常有帮助。

所以，让我们先看看我们的投票应用的状态树应该是什么样的。应用的目标是可以针对多个选项进行投票，那么符合直觉的一种初始化状态应该是包含要被投票的选项集合，我们称之为条目[entries]：

![](http://teropa.info/images/vote_server_tree_entries.png)

当投票开始，还必须定位哪些选项是当前项。所以我们可能还需要一个vote条目，它用来存储当前投票的数据对，投票项应该是来自entries中的：

![](http://teropa.info/images/vote_server_tree_pair.png)

除此之外，投票的计数也应该被保存起来：

![](http://teropa.info/images/vote_server_tree_tally.png)

每次用户进行二选一后，未被选择的那项直接丢弃，被选择的条目重新放回entries的末尾，然后从entries头部选择下一对投票项：

![](http://teropa.info/images/vote_server_tree_next.png)

我们可以想象一下，这么周而复始的投票，最终将会得到一个结果，投票也就结束了：

![](http://teropa.info/images/vote_server_tree_winner.png)

如此设计看起来是合情合理的。针对上面的场景存在很多不同的设计，我们当前的做法也可能不是最佳的，但我们暂时就先这么定吧，足够我们进行下一步了。最重要的是我们在没有写任何代码的前提下已经从最初的点子过渡到确定了应用的具体功能。

### 4.2 项目安排

是时候开始脏活累活了。开始之前，我们先创建一个项目目录：

	mkdir voting-server
	cd voting-server
	npm init         #所有提示问题直接敲回车即可

初始化完毕后，我们的项目目录下将会只存在一个*package.json*文件。

我们将采用ES6语法来写代码。Node是从4.0.0版本后开始支持大多数ES6语法的，并且目前并不支持modules，但我们需要用到。我们将加入Babel，这样我们就能将ES6直接转换成ES5了：

	npm install --save-dev babel

我们还需要些库来用于写单元测试：

	npm install --save-dev mocha chai

[Mocha](https://mochajs.org/)是一个我们将要使用的测试框架，[Chai](http://chaijs.com/)是一个我们用来测试的断言库。

我们将使用下面的mocha命令来跑测试项：

	./node_modules/mocha/bin/mocha --compilers js:babel/register --recursive

这条命令告诉Mocha递归的去项目中查找并执行所有测试项，但执行前先使用Babel进行语法转换。

为了使用方便，可以在我们的*package.json*中添加下面这段代码：

	"scripts": {
  		"test": "mocha --compilers js:babel/register --recursive"
	},

这样以后我们跑测试就只需要执行：

	npm run test

另外，我们还可以添加*test:watch*命令，它用来监控文件变化并自动跑测试项：

	"scripts": {
  		"test": "mocha --compilers js:babel/register --recursive",
  		"test:watch": "npm run test -- --watch"
	},

我们还将用到一个库，来自于facebook：[Immutable](http://facebook.github.io/immutable-js/)，它提供了许多数据结构供我们使用。下一小节我们再来讨论Immutable，但我们在这里先将它加入到我们的项目中，附带[chai-immutable](https://github.com/astorije/chai-immutable)库，它用来向Chai库加入不可变数据结构比对功能：

	npm install --save immutable
	npm install --save-dev chai-immutable

我们需要在所有测试代码前先加入chai-immutable插件，所以我们来先创建一个测试辅助文件：

	//test/test_helper.js

	import chai from 'chai';
	import chaiImmutable from 'chai-immutable';

	chai.use(chaiImmutable);

然后我们需要让Mocha在开始跑测试之前先加载这个文件，修改package.json：

	"scripts": {
  		"test": "mocha --compilers js:babel/register --		require ./test/test_helper.js  --recursive",
  		"test:watch": "npm run test -- --watch"
	},

好了，准备的差不多了。

### 4.3 关于Immutable

第二个值得重视的点是，Redux架构下状态并非只是一个普通的tree，而是一棵不可变的tree。

回想一下前面我们设计的状态tree，你可能会觉得可以直接在应用的代码里直接更新tree：修改映射的值，或删除数组元素等。然而，这并不是Redux允许的。

一个Redux应用的状态树是不可变的数据结构。这意味着，一旦你得到了一棵状态树，它就不会在改变了。任何用户行为改变应用状态，你都会获取一棵映射应用改变后新状态的完整状态树。

这说明任何连续的状态（改变前后）都被分别存储在独立的两棵树。你通过调用一个函数来从一种状态转入下一个状态。

![](http://teropa.info/images/vote_state_succession.png)

这么做好在哪呢？第一，用户通常想一个undo功能，当你误操作导致破坏了应用状态后，你往往想退回到应用的历史状态，而单一的状态tree让该需求变得廉价，你只需要简单保存上一个状态tree的数据即可。你也可以序列化tree并存储起来以供将来重放，这对debug很有帮助的。

抛开其它的特性不谈，不可变数据至少会让你的代码变得简单，这非常重要。你可以用纯函数来进行编程：接受参数数据，返回数据，其它啥都不做。这种函数拥有可预见性，你可以多次调用它，只要参数一致，它总返回相同的结果（冪等性）。测试将变的容易，你不需要在测试前创建太多的准备，仅仅是传入参数和返回值。

不可变数据结构是我们创建应用状态的基础，让我们花点时间来写一些测试项来保证它的正常工作。

为了更了解不可变性，我们来看一个十分简单的数据结构：假设我们有一个计数应用，它只包含一个计数器变量，该变量会从0增加到1，增加到2，增加到3，以此类推。

如果用不可变数据来设计这个计数器变量，则每当计数器自增，我们不是去改变变量本身。你可以想象成该计数器变量没有“setters”方法，你不能执行`42.setValue(43)`。

每当变化发生，我们将获得一个新的变量，它的值是之前的那个变量的值加1等到的。我们可以为此写一个纯函数，它接受一个参数代表当前的状态，并返回一个值表示新的状态。记住，调用它并会修改传入参数的值。这里看一下函数实现和测试代码：

	//test/immutable_spec.js

	import {expect} from 'chai';

	describe('immutability', () => {

  		describe('a number', () => {

    		function increment(currentState) {
      			return currentState + 1;
    		}

    		it('is immutable', () => {
      			let state = 42;
      			let nextState = increment(state);

      			expect(nextState).to.equal(43);
      			expect(state).to.equal(42);
    		});

  		});
	});

可以看到当`increment`调用后`state`并没有被修改，这是因为`Numbers`是不可变的。

我们接下来要做的是让各种数据结构都不可变，而不仅仅是一个整数。

利用Immutable提供的[Lists](https://facebook.github.io/immutable-js/docs/#/Listf)，我们可以假设我们的应用拥有一个电影列表的状态，并且有一个操作用来向当前列表中添加新电影，新列表数据是添加前的列表数据和新增的电影条目合并后的结果，注意，添加前的旧列表数据并没有被修改哦：

	//test/immutable_spec.json

	import {expect} from 'chai';
	import {List} from 'immutable';

	describe('immutability', () => {

  		// ...

  		describe('A List', () => {

    		function addMovie(currentState, movie) {
      			return currentState.push(movie);
    		}

    		it('is immutable', () => {
      			let state = List.of('Trainspotting', '28 Days Later');
      			let nextState = addMovie(state, 'Sunshine');

      			expect(nextState).to.equal(List.of(
        			'Trainspotting',
        			'28 Days Later',
        			'Sunshine'
      			));
      			expect(state).to.equal(List.of(
        			'Trainspotting',
        			'28 Days Later'
      			));
    		});
  		});
	});

如果我们使用的是原生态js数组，那么上面的`addMovie`函数并不会保证旧的状态不会被修改。这里我们使用的是Immutable List。

真实软件中，一个状态树通常是嵌套了多种数据结构的：list，map以及其它类型的集合。假设状态树是一个包含了*movies*列表的hash map，添加一个电影意味着我们需要创建一个新的map，并且在新的map的*movies*元素中添加该新增数据：

	//test/immutable_spec.json

	import {expect} from 'chai';
	import {List, Map} from 'immutable';

	describe('immutability', () => {

  		// ...

  		describe('a tree', () => {

    		function addMovie(currentState, movie) {
      			return currentState.set(
        			'movies',
       				 currentState.get('movies').push(movie)
      			);
    		}

    		it('is immutable', () => {
      			let state = Map({
        			movies: List.of('Trainspotting', '28 Days Later')
      			});
      			let nextState = addMovie(state, 'Sunshine');

      			expect(nextState).to.equal(Map({
        			movies: List.of(
          				'Trainspotting',
          				'28 Days Later',
          				'Sunshine'
        			)
      			}));
      			expect(state).to.equal(Map({
       	 			movies: List.of(
          				'Trainspotting',
          				'28 Days Later'
        			)
      			}));
    		});
  		});
	});

该例子和前面的那个类似，主要用来展示在嵌套结构下Immutable的行为。

针对类似上面这个例子的嵌套数据结构，Immutable提供了很多辅助函数，可以帮助我们更容易的定位嵌套数据的内部属性，以达到更新对应值的目的。我们可以使用一个叫`update`的方法来修改上面的代码：

	//test/immutable_spec.json

	function addMovie(currentState, movie) {
  		return currentState.update('movies', movies => movies.push(movie));
	}

现在我们很好的了解了不可变数据，这将被用于我们的应用状态。[Immutable API](https://facebook.github.io/immutable-js/docs/#/)提供了非常多的辅助函数，我们目前只是学了点皮毛。

不可变数据是Redux的核心理念，但并不是必须使用Immutable库来实现这个特性。事实上，[官方Redux文档](http://rackt.github.io/redux/)使用的是原生js对象和数组，并通过简单的扩展它们来实现的。

这个教程中，我们将使用Immutable库，原因如下：

- 该库将使得实现不可变数据结构变得非常简单；
- 我个人偏爱于将尽可能的使用不可变数据，如果你的数据允许直接修改，迟早会有人踩坑；
- 不可变数据结构更新是持续的，意味着很容易产生性能平静，特别维护是非常庞大的状态树，使用原生js对象和数组意味着要频繁的进行拷贝，很容易导致性能问题。

### 4.4 基于纯函数实现应用逻辑

根据目前我们掌握的不可变状态树和相关操作，我们可以尝试实现投票应用的逻辑。应用的核心逻辑我们拆分成：状态树结构和生成新状态树的函数集合。

#### 4.4.1 加载条目

首先，之前说到，应用允许“加载”一个用来投票的条目集。我们需要一个`setEntries`函数，它用来提供应用的初始化状态：

	//test/core_spec.js

	import {List, Map} from 'immutable';
	import {expect} from 'chai';

	import {setEntries} from '../src/core';

	describe('application logic', () => {

	  describe('setEntries', () => {

	    it('adds the entries to the state', () => {
	      const state = Map();
	      const entries = List.of('Trainspotting', '28 Days Later');
	      const nextState = setEntries(state, entries);
	      expect(nextState).to.equal(Map({
	        entries: List.of('Trainspotting', '28 Days Later')
	      }));
	    });
	  });
	});

我们目前`setEntries`函数的第一版非常简单：在状态map中创建一个`entries`键，并设置给定的条目List。

	//src/core.js

	export function setEntries(state, entries) {
		return state.set('entries', entries);
	}

为了方便起见，我们允许函数第二个参数接受一个原生js数组（或支持iterable的类型），但在状态树中它应该是一个Immutable List：

	//test/core_spec.js

	it('converts to immutable', () => {
	  const state = Map();
	  const entries = ['Trainspotting', '28 Days Later'];
	  const nextState = setEntries(state, entries);
	  expect(nextState).to.equal(Map({
	    entries: List.of('Trainspotting', '28 Days Later')
	  }));
	});

为了达到要求，我们需要修改一下代码：

	//src/core.js

	import {List} from 'immutable';

	export function setEntries(state, entries) {
	  return state.set('entries', List(entries));
	}

#### 4.4.2 开始投票

当state加载了条目集合后，我们可以调用一个`next`函数来开始投票。这表示，我们到了之前设计的状态树的第二阶段。

`next`函数需要在状态树创建中一个投票map，该map有拥有一个`pair`键，值为投票条目中的前两个元素。
这两个元素一旦确定，就要从之前的条目列表中清除：

	//test/core_spec.js

	import {List, Map} from 'immutable';
	import {expect} from 'chai';
	import {setEntries, next} from '../src/core';

	describe('application logic', () => {

	  // ..

	  describe('next', () => {

	    it('takes the next two entries under vote', () => {
	      const state = Map({
	        entries: List.of('Trainspotting', '28 Days Later', 'Sunshine')
	      });
	      const nextState = next(state);
	      expect(nextState).to.equal(Map({
	        vote: Map({
	          pair: List.of('Trainspotting', '28 Days Later')
	        }),
	        entries: List.of('Sunshine')
	      }));
	    });
	  });
	});

`next`函数实现如下：

	//src/core.js

	import {List, Map} from 'immutable';

	// ...

	export function next(state) {
	  const entries = state.get('entries');
	  return state.merge({
	    vote: Map({pair: entries.take(2)}),
	    entries: entries.skip(2)
	  });
	}

#### 4.4.3 投票

当用户产生投票行为后，每当用户给某个条目投了一票后，`vote`将会为这个条目添加`tally`信息，如果对应的
条目信息已存在，则需要则增：

	//test/core_spec.js

	import {List, Map} from 'immutable';
	import {expect} from 'chai';
	import {setEntries, next, vote} from '../src/core';

	describe('application logic', () => {

	  // ...

	  describe('vote', () => {

	    it('creates a tally for the voted entry', () => {
	      const state = Map({
	        vote: Map({
	          pair: List.of('Trainspotting', '28 Days Later')
	        }),
	        entries: List()
	      });
	      const nextState = vote(state, 'Trainspotting');
	      expect(nextState).to.equal(Map({
	        vote: Map({
	          pair: List.of('Trainspotting', '28 Days Later'),
	          tally: Map({
	            'Trainspotting': 1
	          })
	        }),
	        entries: List()
	      }));
	    });

	    it('adds to existing tally for the voted entry', () => {
	      const state = Map({
	        vote: Map({
	          pair: List.of('Trainspotting', '28 Days Later'),
	          tally: Map({
	            'Trainspotting': 3,
	            '28 Days Later': 2
	          })
	        }),
	        entries: List()
	      });
	      const nextState = vote(state, 'Trainspotting');
	      expect(nextState).to.equal(Map({
	        vote: Map({
	          pair: List.of('Trainspotting', '28 Days Later'),
	          tally: Map({
	            'Trainspotting': 4,
	            '28 Days Later': 2
	          })
	        }),
	        entries: List()
	      }));
	    });
	  });
	});

为了让上面的测试项通过，我们可以如下实现`vote`函数：

	//src/core.js

	export function vote(state, entry) {
	  return state.updateIn(
	    ['vote', 'tally', entry],
	    0,
	    tally => tally + 1
	  );
	}

[updateIn](https://facebook.github.io/immutable-js/docs/#/Map/updateIn)让我们更容易完成目标。
它接受的第一个参数是个表达式，含义是“定位到嵌套数据结构的指定位置，路径为：['vote', 'tally', 'Trainspotting']”，
并且执行后面逻辑：如果路径指定的位置不存在，则创建新的映射对，并初始化为0，否则对应值加1。

可能对你来说上面的语法太过于晦涩，但一旦你掌握了它，你将会发现用起来非常的酸爽，所以花一些时间学习并
适应它是非常值得的。

#### 4.4.4 继续投票

每次完成一次二选一投票，用户将进入到第二轮投票，每次得票最高的选项将被保存并添加回条目集合。我们需要添加
这个逻辑到`next`函数中：

	//test/core_spec.js

	describe('next', () => {

	  // ...

	  it('puts winner of current vote back to entries', () => {
	    const state = Map({
	      vote: Map({
	        pair: List.of('Trainspotting', '28 Days Later'),
	        tally: Map({
	          'Trainspotting': 4,
	          '28 Days Later': 2
	        })
	      }),
	      entries: List.of('Sunshine', 'Millions', '127 Hours')
	    });
	    const nextState = next(state);
	    expect(nextState).to.equal(Map({
	      vote: Map({
	        pair: List.of('Sunshine', 'Millions')
	      }),
	      entries: List.of('127 Hours', 'Trainspotting')
	    }));
	  });

	  it('puts both from tied vote back to entries', () => {
	    const state = Map({
	      vote: Map({
	        pair: List.of('Trainspotting', '28 Days Later'),
	        tally: Map({
	          'Trainspotting': 3,
	          '28 Days Later': 3
	        })
	      }),
	      entries: List.of('Sunshine', 'Millions', '127 Hours')
	    });
	    const nextState = next(state);
	    expect(nextState).to.equal(Map({
	      vote: Map({
	        pair: List.of('Sunshine', 'Millions')
	      }),
	      entries: List.of('127 Hours', 'Trainspotting', '28 Days Later')
	    }));
	  });
	});

我们需要一个`getWinners`函数来帮我们选择谁是赢家：

	//src/core.js

	function getWinners(vote) {
	  if (!vote) return [];
	  const [a, b] = vote.get('pair');
	  const aVotes = vote.getIn(['tally', a], 0);
	  const bVotes = vote.getIn(['tally', b], 0);
	  if      (aVotes > bVotes)  return [a];
	  else if (aVotes < bVotes)  return [b];
	  else                       return [a, b];
	}

	export function next(state) {
	  const entries = state.get('entries')
	                       .concat(getWinners(state.get('vote')));
	  return state.merge({
	    vote: Map({pair: entries.take(2)}),
	    entries: entries.skip(2)
	  });
	}

#### 4.4.5 投票结束

当投票项只剩一个时，投票结束：

	//test/core_spec.js

	describe('next', () => {

	  // ...

	  it('marks winner when just one entry left', () => {
	    const state = Map({
	      vote: Map({
	        pair: List.of('Trainspotting', '28 Days Later'),
	        tally: Map({
	          'Trainspotting': 4,
	          '28 Days Later': 2
	        })
	      }),
	      entries: List()
	    });
	    const nextState = next(state);
	    expect(nextState).to.equal(Map({
	      winner: 'Trainspotting'
	    }));
	  });
	});

我们需要在`next`函数中增加一个条件分支，用来匹配上面的逻辑：

	//src/core.js

	export function next(state) {
	  const entries = state.get('entries')
	                       .concat(getWinners(state.get('vote')));
	  if (entries.size === 1) {
	    return state.remove('vote')
	                .remove('entries')
	                .set('winner', entries.first());
	  } else {
	    return state.merge({
	      vote: Map({pair: entries.take(2)}),
	      entries: entries.skip(2)
	    });
	  }
	}

我们可以直接返回`Map({winner: entries.first()})`，但我们还是基于旧的状态数据进行一步一步的
操作最终得到结果，这么做是为将来做打算。因为应用将来可能还会有很多其它状态数据在Map中，这是一个写测试项的好习惯。
所以我们以后要记住，不要重新创建一个状态数据，而是从旧的状态数据中生成新的状态实例。

到此为止我们已经有了一套可以接受的应用核心逻辑实现，表现形式为几个独立的函数。我们也有针对这些函数的
测试代码，这些测试项很容易写：No setup, no mocks, no stubs。这就是纯函数的魅力，我们只需要调用它们，
并检查返回值就行了。

提醒一下，我们目前还没有安装redux哦，我们就已经可以专注于应用自身的逻辑本身进行实现，而不被所谓的框架
所干扰。这真的很不错，对吧？

### 4.5 初识Actions和Reducers

我们有了应用的核心函数，但在Redux中我们不应该直接调用函数。在这些函数和应用之间还存在这一个中间层：Actions。

Action是一个描述应用状态变化发生的简单数据结构。按照约定，每个action都包含一个`type`属性，
该属性用于描述操作类型。action通常还包含其它属性，下面是一个简单的action例子，该action用来匹配
前面我们写的业务操作：

	{type: 'SET_ENTRIES', entries: ['Trainspotting', '28 Days Later']}

	{type: 'NEXT'}

	{type: 'VOTE', entry: 'Trainspotting'}

actions的描述就这些，但我们还需要一种方式用来把它绑定到我们实际的核心函数上。举个例子：

	// 定义一个action
	let voteAction = {type: 'VOTE', entry: 'Trainspotting'}
	// 该action应该触发下面的逻辑
	return vote(state, voteAction.entry);

我们接下来要用到的是一个普通函数，它用来根据action和当前state来调用指定的核心函数，我们称这种函数叫：
reducer：

	//src/reducer.js

	export default function reducer(state, action) {
	  // Figure out which function to call and call it
	}

我们应该测试这个reducer是否可以正确匹配我们之前的三个actions：

	//test/reducer_spec.js

	import {Map, fromJS} from 'immutable';
	import {expect} from 'chai';

	import reducer from '../src/reducer';

	describe('reducer', () => {

	  it('handles SET_ENTRIES', () => {
	    const initialState = Map();
	    const action = {type: 'SET_ENTRIES', entries: ['Trainspotting']};
	    const nextState = reducer(initialState, action);

	    expect(nextState).to.equal(fromJS({
	      entries: ['Trainspotting']
	    }));
	  });

	  it('handles NEXT', () => {
	    const initialState = fromJS({
	      entries: ['Trainspotting', '28 Days Later']
	    });
	    const action = {type: 'NEXT'};
	    const nextState = reducer(initialState, action);

	    expect(nextState).to.equal(fromJS({
	      vote: {
	        pair: ['Trainspotting', '28 Days Later']
	      },
	      entries: []
	    }));
	  });

	  it('handles VOTE', () => {
	    const initialState = fromJS({
	      vote: {
	        pair: ['Trainspotting', '28 Days Later']
	      },
	      entries: []
	    });
	    const action = {type: 'VOTE', entry: 'Trainspotting'};
	    const nextState = reducer(initialState, action);

	    expect(nextState).to.equal(fromJS({
	      vote: {
	        pair: ['Trainspotting', '28 Days Later'],
	        tally: {Trainspotting: 1}
	      },
	      entries: []
	    }));
	  });
	});

我们的reducer将根据action的type来选择对应的核心函数，它同时也应该知道如何使用action的额外属性：

	//src/reducer.js

	import {setEntries, next, vote} from './core';

	export default function reducer(state, action) {
	  switch (action.type) {
	  case 'SET_ENTRIES':
	    return setEntries(state, action.entries);
	  case 'NEXT':
	    return next(state);
	  case 'VOTE':
	    return vote(state, action.entry)
	  }
	  return state;
	}

注意，如果reducer没有匹配到action，则应该返回当前的state。

reducers还有一个需要特别注意的地方，那就是当传递一个未定义的state参数时，reducers应该知道如何
初始化state为有意义的值。我们的场景中，初始值为Map，因此如果传给reducer一个`undefined`state的话，
reducers将使用一个空的Map来代替：

	//test/reducer_spec.js

	describe('reducer', () => {

	  // ...

	  it('has an initial state', () => {
	    const action = {type: 'SET_ENTRIES', entries: ['Trainspotting']};
	    const nextState = reducer(undefined, action);
	    expect(nextState).to.equal(fromJS({
	      entries: ['Trainspotting']
	    }));
	  });
	});

之前在我们的`cores.js`文件中，我们定义了初始值：

	//src/core.js

	export const INITIAL_STATE = Map();

所以在reducer中我们可以直接导入它：

	//src/reducer.js

	import {setEntries, next, vote, INITIAL_STATE} from './core';

	export default function reducer(state = INITIAL_STATE, action) {
	  switch (action.type) {
	  case 'SET_ENTRIES':
	    return setEntries(state, action.entries);
	  case 'NEXT':
	    return next(state);
	  case 'VOTE':
	    return vote(state, action.entry)
	  }
	  return state;
	}

事实上，提供一个action集合，你可以将它们分解并作用在当前状态上，这也是为什么称它们为reducer的原因：
它完全适配reduce方法：

	//test/reducer_spec.js

	it('can be used with reduce', () => {
	  const actions = [
	    {type: 'SET_ENTRIES', entries: ['Trainspotting', '28 Days Later']},
	    {type: 'NEXT'},
	    {type: 'VOTE', entry: 'Trainspotting'},
	    {type: 'VOTE', entry: '28 Days Later'},
	    {type: 'VOTE', entry: 'Trainspotting'},
	    {type: 'NEXT'}
	  ];
	  const finalState = actions.reduce(reducer, Map());

	  expect(finalState).to.equal(fromJS({
	    winner: 'Trainspotting'
	  }));
	});

相比直接调用核心业务函数，这种批处理或称之为重放一个action集合的能力主要依赖于状态转换的action/reducer模型。
举个例子，你可以把actions序列化成json，并轻松的将它发送给Web Worker去执行你的reducer逻辑。或者
直接通过网络发送到其它地方供日后执行！

注意我们这里使用的是普通js对象作为actions，而并非不可变数据类型。这是Redux提倡我们的做法。

### 4.6 尝试Reducer协作

目前我们的核心函数都是接受整个state并返回更新后的整个state。

这么做在大型应用中可能并不太明智。如果你的应用所有操作都要求必须接受完整的state，那么这个项目维护起来就是灾难。
日后如果你想进行state结构的调整，你将会付出惨痛的代价。

其实有更好的做法，你只需要保证组件操作尽可能小的state片段即可。我们这里提到的就是模块化思想：
提供给模块仅它需要的数据，不多不少。

我们的应用很小，所以这并不是太大的问题，但我们还是选择改善这一点：没有必要给`vote`函数传递整个state，它只需要`vote`
部分。让我们修改一下对应的测试代码：

	//test/core_spec.js

	describe('vote', () => {

	  it('creates a tally for the voted entry', () => {
	    const state = Map({
	      pair: List.of('Trainspotting', '28 Days Later')
	    });
	    const nextState = vote(state, 'Trainspotting')
	    expect(nextState).to.equal(Map({
	      pair: List.of('Trainspotting', '28 Days Later'),
	      tally: Map({
	        'Trainspotting': 1
	      })
	    }));
	  });

	  it('adds to existing tally for the voted entry', () => {
	    const state = Map({
	      pair: List.of('Trainspotting', '28 Days Later'),
	      tally: Map({
	        'Trainspotting': 3,
	        '28 Days Later': 2
	      })
	    });
	    const nextState = vote(state, 'Trainspotting');
	    expect(nextState).to.equal(Map({
	      pair: List.of('Trainspotting', '28 Days Later'),
	      tally: Map({
	        'Trainspotting': 4,
	        '28 Days Later': 2
	      })
	    }));
	  });
	});

看，测试代码更加简单了。

`vote`函数的实现也需要更新：

	//src/core.js

	export function vote(voteState, entry) {
	  return voteState.updateIn(
	    ['tally', entry],
	    0,
	    tally => tally + 1
	  );
	}

最后我们还需要修改`reducer`，只传递需要的state给`vote`函数：

	//src/reducer.js

	export default function reducer(state = INITIAL_STATE, action) {
	  switch (action.type) {
	  case 'SET_ENTRIES':
	    return setEntries(state, action.entries);
	  case 'NEXT':
	    return next(state);
	  case 'VOTE':
	    return state.update('vote',
	                        voteState => vote(voteState, action.entry));
	  }
	  return state;
	}

这个做法在大型项目中非常重要：根reducer只传递部分state给下一级reducer。我们将定位合适的state片段的工作
从对应的更新操作中分离出来。

[Redux的reducers文档](http://rackt.github.io/redux/docs/basics/Reducers.html)针对这一细节
介绍了更多内容，并描述了一些辅助函数的用法，可以在更多长场景中有效的使用。

### 4.7 初识Redux Store

现在我们可以开始了解如何将上面介绍的内容使用在Redux中了。

如你所见，如果你有一个actions集合，你可以调用`reduce`，获得最终的应用状态。当然，通常情况下不会如此，actions
将会在不同的时间发生：用户操作，远程调用，超时触发器等。

针对这些情况，我们可以使用Redux Store。从名字可以看出它用来存储应用的状态。

Redux Store通常会由一个reducer函数初始化，如我们之前实现的：

	import {createStore} from 'redux';

	const store = createStore(reducer);

接下来你就可以向这个Store指派actions了。Store内部将会使用你实现的reducer来处理action，并负责传递给
reducer应用的state，最后负责存储reducer返回的新state：

	store.dispatch({type: 'NEXT'});

任何时刻你都可以通过下面的方法获取当前的state：

	store.getState();

我们将会创建一个`store.js`用来初始化和导出一个Redux Store对象。让我们先写测试代码吧：

	//test/store_spec.js

	import {Map, fromJS} from 'immutable';
	import {expect} from 'chai';

	import makeStore from '../src/store';

	describe('store', () => {

	  it('is a Redux store configured with the correct reducer', () => {
	    const store = makeStore();
	    expect(store.getState()).to.equal(Map());

	    store.dispatch({
	      type: 'SET_ENTRIES',
	      entries: ['Trainspotting', '28 Days Later']
	    });
	    expect(store.getState()).to.equal(fromJS({
	      entries: ['Trainspotting', '28 Days Later']
	    }));
	  });
	});

在创建Store之前，我们先在项目中加入Redux库：

	npm install --save redux

然后我们新建`store.js`文件，如下：

	//src/store.js

	import {createStore} from 'redux';
	import reducer from './reducer';

	export default function makeStore() {
	  return createStore(reducer);
	}

Redux Store负责将应用的所有组件关联起来：它持有应用的当前状态，并负责指派actions，且负责调用包含了
业务逻辑的reducer。

应用的业务代码和Redux的整合方式非常引人注目，因为我们只有一个普通的reducer函数，这是唯一需要告诉Redux
的事儿。其它部分全部都是我们自己的，没有框架入侵的，高便携的纯函数代码！

现在我们创建一个应用的入口文件`index.js`：

	//index.js

	import makeStore from './src/store';

	export const store = makeStore();

现在我们可以开启一个[Node REPL](http://segmentfault.com/a/1190000002673137)（例如babel-node）,
载入`index.js`文件来测试执行了。

### 4.8 配置Socket.io服务

我们的应用服务端用来为一个提供投票和显示结果浏览器端提供服务的，为了这个目的，我们需要考虑两端通信的方式。

这个应用需要实时通信，这确保我们的投票者可以实时查看到所有人的投票信息。为此，我们选择使用WebSockets作为
通信方式。因此，我们选择[Socket.io](http://socket.io/)库作为跨终端的websocket抽象实现层，它在客户端
不支持websocket的情况下提供了多种备选方案。

让我们在项目中加入Socket.io：

	npm install --save socket.io

现在，让我新建一个`server.js`文件：

	//src/server.js

	import Server from 'socket.io';

	export default function startServer() {
	const io = new Server().attach(8090);
	}

这里我们创建了一个Socket.io 服务，绑定8090端口。端口号是我随意选的，你可以更改，但后面客户端连接时
要注意匹配。

现在我们可以在`index.js`中调用这个函数：

	//index.js

	import makeStore from './src/store';
	import startServer from './src/server';

	export const store = makeStore();
	startServer();

我们现在可以在`package.json`中添加`start`指令来方便启动应用：

	//package.json
	"scripts": {
		"start": "babel-node index.js",
		"test": "mocha --compilers js:babel/register  --require ./test/test_helper.js  --recursive",
		"test:watch": "npm run test --watch"
	},

这样我们就可以直接执行下面命令来开启应用：

	npm run start

### 4.9 用Redux监听器传播State

我们现在拥有了一个Socket.io服务，也建立了Redux状态容器，但它们并没有整合在一起，这就是我们接下来要做的事儿。

我们的服务端需要让客户端知道当前的应用状态（例如：“正在投票的项目是什么？”，“当前的票数是什么？”，
“已经出来结果了吗？”）。这些都可以通过每当变化发生时[触发Socket.io事件](http://socket.io/docs/server-api/#server#emit)来实现。

我们如何得知什么时候发生变化？Redux对此提供了方案：你可以订阅Redux Store。这样每当store指派了action之后，在可能发生变化前
会调用你提供的指定回调函数。

我们要修改一下`startServer`实现，我们先来调整一下index.js：

	//index.js

	import makeStore from './src/store';
	import {startServer} from './src/server';

	export const store = makeStore();
	startServer(store);

接下来我们只需监听store的状态，并把它序列化后用socket.io事件传播给所有处于连接状态的客户端。

	//src/server.js

	import Server from 'socket.io';

	export function startServer(store) {
	  const io = new Server().attach(8090);

	  store.subscribe(
	    () => io.emit('state', store.getState().toJS())
	  );
	}

目前我们的做法是一旦状态有改变，就发送整个state给所有客户端，很容易想到这非常不友好，产生大量流量
损耗，更好的做法是只传递改变的state片段，但我们为了简单，在这个例子中就先这么实现吧。

除了状态发生变化时发送状态数据外，每当新客户端连接服务器端时也应该直接发送当前的状态给该客户端。

我们可以通过监听Socket.io的`connection`事件来实现上述需求：

	//src/server.js

	import Server from 'socket.io';

	export function startServer(store) {
	  const io = new Server().attach(8090);

	  store.subscribe(
	    () => io.emit('state', store.getState().toJS())
	  );

	  io.on('connection', (socket) => {
	    socket.emit('state', store.getState().toJS());
	  });
	}

### 4.10 接受远程调用Redux Actions

除了将应用状态同步给客户端外，我们还需要接受来自客户端的更新操作：投票者需要发起投票，投票组织者需要
发起下一轮投票的请求。

我们的解决方案非常简单。我们只需要让客户端发布“action”事件即可，然后我们直接将事件发送给Redux Store：

	//src/server.js

	import Server from 'socket.io';

	export function startServer(store) {
	  const io = new Server().attach(8090);

	  store.subscribe(
	    () => io.emit('state', store.getState().toJS())
	  );

	  io.on('connection', (socket) => {
	    socket.emit('state', store.getState().toJS());
	    socket.on('action', store.dispatch.bind(store));
	  });
	}

这样我们就完成了远程调用actions。Redux架构让我们的项目更加简单：actions仅仅是js对象，可以很容易用于
网络传输，我们现在实现了一个支持多人投票的服务端系统，很有成就感吧。

现在我们的服务端操作流程如下：

1. 客户端发送一个action给服务端；
2. 服务端交给Redux Store处理action；
3. Store调用reducer，reducer执行对应的应用逻辑；
4. Store根据reducer的返回结果来更新状态；
5. Store触发服务端监听的回调函数；
6. 服务端触发“state”事件；
7. 所有连接的客户端接受到新的状态。

在结束服务端开发之前，我们载入一些测试数据来感受一下。我们可以添加`entries.json`文件：

	//entries.json

	[
	  "Shallow Grave",
	  "Trainspotting",
	  "A Life Less Ordinary",
	  "The Beach",
	  "28 Days Later",
	  "Millions",
	  "Sunshine",
	  "Slumdog Millionaire",
	  "127 Hours",
	  "Trance",
	  "Steve Jobs"
	]

我们在`index.json`中加载它然后发起`next`action来开启投票：

	//index.js

	import makeStore from './src/store';
	import {startServer} from './src/server';

	export const store = makeStore();
	startServer(store);

	store.dispatch({
	  type: 'SET_ENTRIES',
	  entries: require('./entries.json')
	});
	store.dispatch({type: 'NEXT'});

那么接下来我们就来看看如何实现客户端。

## 5.开发客户端应用

本教程剩余的部分就是写一个React应用，用来连接服务端，并提供投票给使用者。

在客户端我们依然使用Redux。这是更常见的搭配：用于React应用的底层引擎。我们已经了解到Redux如何使用。
现在我们将学习它是如何结合并影响React应用的。

我推荐大家跟随本教程的步骤完成应用，但你也可以从[github](https://github.com/teropa/redux-voting-client)上获取源码。

### 5.1 客户端项目创建

第一件事儿我们当然是创建一个新的NPM项目，如下：

	mkdir voting-client
	cd voting-client
	npm init            # Just hit enter for each question

我们的应用需要一个html主页，我们放在`dist/index.html`：

	//dist/index.html

	<!DOCTYPE html>
	<html>
	<body>
	  <div id="app"></div>
	  <script src="bundle.js"></script>
	</body>
	</html>

这个页面包含一个id为app的`<div>`，我们将在其中插入我们的应用。在同级目录下还需要一个`bundle.js`文件。

我们为应用新建第一个js文件，它是系统的入口文件。目前我们先简单的添加一行日志代码：

	//src/index.js
	console.log('I am alive!');

为了给我们客户端开发减负，我们将使用[Webpack](http://webpack.github.io/)，让我们加入到项目中：

	npm install --save-dev webpack webpack-dev-server

接下来，我们在项目根目录新建一个Webpack配置文件：

	//webpack.config.js

	module.exports = {
	  entry: [
	    './src/index.js'
	  ],
	  output: {
	    path: __dirname + '/dist',
	    publicPath: '/',
	    filename: 'bundle.js'
	  },
	  devServer: {
	    contentBase: './dist'
	  }
	};

配置表明将找到我们的`index.js`入口，并编译到`dist/bundle.js`中。同时把`dist`目录当作开发服务器根目录。

你现在可以执行Webpack来生成`bundle.js`：

	webpack

你也可以开启一个开发服务器，访问localhost:8080来测试页面效果：

	webpack-dev-server

由于我们将使用ES6语法和React的[JSX语法](https://facebook.github.io/jsx/)，我们需要一些工具。
Babel是一个非常合适的选择，我们需要Babel库：

	npm install --save-dev babel-core babel-loader

我们可以在Webpack配置文件中添加一些配置，这样webpack将会对`.jsx`和`.js`文件使用Babel进行处理：

	//webpack.config.js

	module.exports = {
		entry: [
			'./src/index.js'
		],
		module: {
			loaders: [{
				test: /\.jsx?$/,
				exclude: /node_modules/,
				loader: 'babel'
			}]
		},
		resolve: {
			extensions: ['', '.js', '.jsx']
		},
		output: {
			path: __dirname + '/dist',
			publicPath: '/',
			filename: 'bundle.js'
		},
		devServer: {
			contentBase: './dist'
		}
	};


### 5.2 单元测试支持

我们也将会为客户端代码编写一些单元测试。我们使用与服务端相同的测试套件：

	npm install --save-dev mocha chai

我们也将会测试我们的React组件，这就要求需要一个DOM库。我们可能需要像[Karma](http://karma-runner.github.io/0.13/index.html)
库一样的功能来进行真实web浏览器测试。但我们这里准备使用一个node端纯js的dom库：

	npm install --save-dev jsdom@3

在用于react之前我们需要一些jsdom的预备代码。我们需要创建通常在浏览器端被提供的`document`和`window`对象。
并且将它们声明为全局对象，这样才能被React使用。我们可以创建一个测试辅助文件做这些工作：

	//test/test_helper.js

	import jsdom from 'jsdom';

	const doc = jsdom.jsdom('<!doctype html><html><body></body></html>');
	const win = doc.defaultView;

	global.document = doc;
	global.window = win;

此外，我们还需要将jsdom提供的`window`对象的所有属性导入到Node.js的全局变量中，这样使用这些属性时
就不需要`window.`前缀，这才满足在浏览器环境下的用法：

	//test/test_helper.js

	import jsdom from 'jsdom';

	const doc = jsdom.jsdom('<!doctype html><html><body></body></html>');
	const win = doc.defaultView;

	global.document = doc;
	global.window = win;

	Object.keys(window).forEach((key) => {
	  if (!(key in global)) {
	    global[key] = window[key];
	  }
	});

我们还需要使用Immutable集合，所以我们也需要参照后段配置添加相应的库：

	npm install --save immutable
	npm install --save-dev chai-immutable

现在我们再次修改辅助文件：

	//test/test_helper.js

	import jsdom from 'jsdom';
	import chai from 'chai';
	import chaiImmutable from 'chai-immutable';

	const doc = jsdom.jsdom('<!doctype html><html><body></body></html>');
	const win = doc.defaultView;

	global.document = doc;
	global.window = win;

	Object.keys(window).forEach((key) => {
	  if (!(key in global)) {
	    global[key] = window[key];
	  }
	});

	chai.use(chaiImmutable);

最后一步是在`package.json`中添加指令：

	//package.json

	"scripts": {
	  "test": "mocha --compilers js:babel-core/register --require ./test/test_helper.js 'test/**/*.@(js|jsx)'"
	},

这几乎和我们在后端做的一样，只有两个地方不同：

- Babel的编译器名称：在该项目中我们使用`babel-core`代替`babel`
- 测试文件设置：服务端我们使用`--recursive`，但这么设置无法匹配`.jsx`文件，所以我们需要使用
[glob](https://github.com/isaacs/node-glob)

为了实现当代码发生修改后自动进行测试，我们依然添加`test:watch`指令：

	//package.json

	"scripts": {
	  "test": "mocha --compilers js:babel-core/register --require ./test/test_helper.js 'test/**/*.@(js|jsx)'",
	  "test:watch": "npm run test -- --watch"
	},

### 5.3 React和react-hot-loader

最后我们来聊聊React！

使用React+Redux+Immutable来开发应用真正酷毙的地方在于：我们可以用纯组件（有时候也称为蠢组件）思想实现
任何东西。这个概念与纯函数很类似，有如下一些规则：

1. 一个纯组件利用props接受所有它需要的数据，类似一个函数的入参，除此之外它不会被任何其它因素影响；
2. 一个纯组件通常没有内部状态。它用来渲染的数据完全来自于输入props，使用相同的props来渲染相同的纯组件多次，
将得到相同的UI。不存在隐藏的内部状态导致渲染不同。

这就带来了[一个和使用纯函数一样的效果](https://www.youtube.com/watch?v=1uRC3hmKQnM&feature=youtu.be&t=13m10s)：
我们可以根据输入来预测一个组件的渲染，我们不需要知道组件的其它信息。这也使得我们的界面测试变得很简单，
与我们测试纯应用逻辑一样简单。

如果组件不包含状态，那么状态放在哪？当然在不可变的Store中啊！我们已经见识过它是怎么运作的了，其
最大的特点就是从界面代码中分离出状态。

在此之前，我们还是先给项目添加React：

	npm install --save react

我们同样需要[react-hot-loader](https://github.com/gaearon/react-hot-loader)。它让我们的开发
变得非常快，因为它提供了我们在不丢失当前状态的情况下重载代码的能力：

	npm install --save-dev react-hot-loader

我们需要更新一下`webpack.config.js`，使其能热加载：

	//webpack.config.js

	var webpack = require('webpack');

	module.exports = {
	  entry: [
	    'webpack-dev-server/client?http://localhost:8080',
	    'webpack/hot/only-dev-server',
	    './src/index.js'
	  ],
	  module: {
	    loaders: [{
	      test: /\.jsx?$/,
	      exclude: /node_modules/,
	      loader: 'react-hot!babel'
	    }],
	  }
	  resolve: {
	    extensions: ['', '.js', '.jsx']
	  },
	  output: {
	    path: __dirname + '/dist',
	    publicPath: '/',
	    filename: 'bundle.js'
	  },
	  devServer: {
	    contentBase: './dist',
	    hot: true
	  },
	  plugins: [
	    new webpack.HotModuleReplacementPlugin()
	  ]
	};

在上述配置的`entry`里我们包含了2个新的应用入口点：webpack dev server和webpack hot module loader。
它们提供了webpack模块热替换能力。该能力并不是默认加载的，所以上面我们才需要在`plugins`和`devServer`
中手动加载。

配置的`loaders`部分我们在原先的Babel前配置了`react-hot`用于`.js`和`.jsx`文件。

如果你现在重启开发服务器，你将看到一个在终端看到Hot Module Replacement已开启的消息提醒。我们可以
开始写我们的第一个组件了。

### 5.4 实现投票界面

应用的投票界面非常简单：一旦投票启动，它将现实2个按钮，分别用来表示2个可选项，当投票结束，它显示最终结果。

![](http://teropa.info/images/voting_shots.png)

我们之前都是以测试先行的开发方式，但是在react组件开发中我们将先实现组件，再进行测试。这是因为
webpack和react-hot-loader提供了更加优良的[反馈机制](http://blog.iterate.no/2012/10/01/know-your-feedback-loop-why-and-how-to-optimize-it/)。
而且，也没有比直接看到界面更加好的测试UI手段了。

让我们假设有一个`Voting`组件，在之前的入口文件`index.html`的`#app`div中加载它。由于我们的代码中
包含JSX语法，所以需要把`index.js`重命名为`index.jsx`：

	//src/index.jsx

	import React from 'react';
	import Voting from './components/Voting';

	const pair = ['Trainspotting', '28 Days Later'];

	React.render(
	  <Voting pair={pair} />,
	  document.getElementById('app')
	);

`Voting`组件将使用`pair`属性来加载数据。我们目前可以先硬编码数据，稍后我们将会用真实数据来代替。
组件本身是纯粹的，并且对数据来源并不敏感。

注意，在`webpack.config.js`中的入口点文件名也要修改：

	//webpack.config.js

	entry: [
	  'webpack-dev-server/client?http://localhost:8080',
	  'webpack/hot/only-dev-server',
	  './src/index.jsx'
	],

如果你此时重启webpack-dev-server，你将看到缺失Voting组件的报错。让我们修复它：

	//src/components/Voting.jsx

	import React from 'react';

	export default React.createClass({
	  getPair: function() {
	    return this.props.pair || [];
	  },
	  render: function() {
	    return <div className="voting">
	      {this.getPair().map(entry =>
	        <button key={entry}>
	          <h1>{entry}</h1>
	        </button>
	      )}
	    </div>;
	  }
	});

你将会在浏览器上看到组件创建的2个按钮。你可以试试修改代码感受一下浏览器自动更新的魅力，没有刷新，
没有页面加载，一切都那么迅雷不及掩耳盗铃。

现在我们来添加第一个单元测试：

	//test/components/Voting_spec.jsx

	import Voting from '../../src/components/Voting';

	describe('Voting', () => {

	});

测试组件渲染的按钮，我们必须先看看它的输出是什么。要在单元测试中渲染一个组件，我们需要`react/addons`提供
的辅助函数[renderIntoDocument](https://facebook.github.io/react/docs/test-utils.html#renderintodocument)：

	//test/components/Voting_spec.jsx

	import React from 'react/addons';
	import Voting from '../../src/components/Voting';

	const {renderIntoDocument} = React.addons.TestUtils;

	describe('Voting', () => {

	  it('renders a pair of buttons', () => {
	    const component = renderIntoDocument(
	      <Voting pair={["Trainspotting", "28 Days Later"]} />
	    );
	  });
	});

一旦组件渲染完毕，我就可以通过react提供的另一个辅助函数[scryRenderedDOMComponentsWithTag](https://facebook.github.io/react/docs/test-utils.html#scryrendereddomcomponentswithtag)
来拿到`button`元素。我们期望存在两个按钮，并且期望按钮的值是我们设置的：

	//test/components/Voting_spec.jsx

	import React from 'react/addons';
	import Voting from '../../src/components/Voting';
	import {expect} from 'chai';

	const {renderIntoDocument, scryRenderedDOMComponentsWithTag}
	  = React.addons.TestUtils;

	describe('Voting', () => {

	  it('renders a pair of buttons', () => {
	    const component = renderIntoDocument(
	      <Voting pair={["Trainspotting", "28 Days Later"]} />
	    );
	    const buttons = scryRenderedDOMComponentsWithTag(component, 'button');

	    expect(buttons.length).to.equal(2);
	    expect(buttons[0].getDOMNode().textContent).to.equal('Trainspotting');
	    expect(buttons[1].getDOMNode().textContent).to.equal('28 Days Later');
	  });
	});

如果我们跑一下测试，将会看到测试通过的提示：

	npm run test

当用户点击某个按钮后，组件将会调用回调函数，该函数也由组件的prop传递给组件。

让我们完成这一步，我们可以通过使用React提供的测试工具[Simulate](https://facebook.github.io/react/docs/test-utils.html#simulate)
来模拟点击操作：

	//test/components/Voting_spec.jsx

	import React from 'react/addons';
	import Voting from '../../src/components/Voting';
	import {expect} from 'chai';

	const {renderIntoDocument, scryRenderedDOMComponentsWithTag, Simulate}
	  = React.addons.TestUtils;

	describe('Voting', () => {

	  // ...

	  it('invokes callback when a button is clicked', () => {
	    let votedWith;
	    const vote = (entry) => votedWith = entry;

	    const component = renderIntoDocument(
	      <Voting pair={["Trainspotting", "28 Days Later"]}
	              vote={vote}/>
	    );
	    const buttons = scryRenderedDOMComponentsWithTag(component, 'button');
	    Simulate.click(buttons[0].getDOMNode());

	    expect(votedWith).to.equal('Trainspotting');
	  });
	});

要想使上面的测试通过很简单，我们只需要让按钮的`onClick`事件调用`vote`并传递选中条目即可：

	//src/components/Voting.jsx

	import React from 'react';

	export default React.createClass({
	  getPair: function() {
	    return this.props.pair || [];
	  },
	  render: function() {
	    return <div className="voting">
	      {this.getPair().map(entry =>
	        <button key={entry}
	                onClick={() => this.props.vote(entry)}>
	          <h1>{entry}</h1>
	        </button>
	      )}
	    </div>;
	  }
	});

这就是我们在纯组件中常用的方式：组件不需要做太多，只是回调传入的参数即可。

注意，这里我们又是先写的测试代码，我发现业务代码的测试要比测试UI更容易写，所以后面我们会保持这种
方式：UI测试后行，业务代码测试先行。

一旦用户已经针对某对选项投过票了，我们就不应该允许他们再次投票，难道我们应该在组件内部维护某种状态么？
不，我们需要保证我们的组件是纯粹的，所以我们需要分离这个逻辑，组件需要一个`hasVoted`属性，我们先硬编码
传递给它：

	//src/index.jsx

	import React from 'react';
	import Voting from './components/Voting';

	const pair = ['Trainspotting', '28 Days Later'];

	React.render(
	  <Voting pair={pair} hasVoted="Trainspotting" />,
	  document.getElementById('app')
	);

我们可以简单的修改一下组件即可：

	//src/components/Voting.jsx

	import React from 'react';

	export default React.createClass({
	  getPair: function() {
	    return this.props.pair || [];
	  },
	  isDisabled: function() {
	    return !!this.props.hasVoted;
	  },
	  render: function() {
	    return <div className="voting">
	      {this.getPair().map(entry =>
	        <button key={entry}
	                disabled={this.isDisabled()}
	                onClick={() => this.props.vote(entry)}>
	          <h1>{entry}</h1>
	        </button>
	      )}
	    </div>;
	  }
	});

让我们再为按钮添加一个提示，当用户投票完毕后，在选中的项目上添加标识，这样用户就更容易理解：

	//src/components/Voting.jsx

	import React from 'react';

	export default React.createClass({
	  getPair: function() {
	    return this.props.pair || [];
	  },
	  isDisabled: function() {
	    return !!this.props.hasVoted;
	  },
	  hasVotedFor: function(entry) {
	    return this.props.hasVoted === entry;
	  },
	  render: function() {
	    return <div className="voting">
	      {this.getPair().map(entry =>
	        <button key={entry}
	                disabled={this.isDisabled()}
	                onClick={() => this.props.vote(entry)}>
	          <h1>{entry}</h1>
	          {this.hasVotedFor(entry) ?
	            <div className="label">Voted</div> :
	            null}
	        </button>
	      )}
	    </div>;
	  }
	});

投票界面最后要添加的，就是获胜者样式。我们可能需要添加新的props：

	//src/index.jsx

	import React from 'react';
	import Voting from './components/Voting';

	const pair = ['Trainspotting', '28 Days Later'];

	React.render(
	  <Voting pair={pair} winner="Trainspotting" />,
	  document.getElementById('app')
	);

我们再次修改一下组件：

	//src/components/Voting.jsx

	import React from 'react';

	export default React.createClass({
	  getPair: function() {
	    return this.props.pair || [];
	  },
	  isDisabled: function() {
	    return !!this.props.hasVoted;
	  },
	  hasVotedFor: function(entry) {
	    return this.props.hasVoted === entry;
	  },
	  render: function() {
	    return <div className="voting">
	      {this.props.winner ?
	        <div ref="winner">Winner is {this.props.winner}!</div> :
	        this.getPair().map(entry =>
	          <button key={entry}
	                  disabled={this.isDisabled()}
	                  onClick={() => this.props.vote(entry)}>
	            <h1>{entry}</h1>
	            {this.hasVotedFor(entry) ?
	              <div className="label">Voted</div> :
	              null}
	          </button>
	        )}
	    </div>;
	  }
	});

目前我们已经完成了所有要做的，但是`render`函数看着有点丑陋，如果我们可以把胜利界面独立成新的组件
可能会好一些：

	//src/components/Winner.jsx

	import React from 'react';

	export default React.createClass({
	  render: function() {
	    return <div className="winner">
	      Winner is {this.props.winner}!
	    </div>;
	  }
	});

这样投票组件就会变得很简单，它只需关注投票按钮逻辑即可：

	//src/components/Vote.jsx

	import React from 'react';

	export default React.createClass({
	  getPair: function() {
	    return this.props.pair || [];
	  },
	  isDisabled: function() {
	    return !!this.props.hasVoted;
	  },
	  hasVotedFor: function(entry) {
	    return this.props.hasVoted === entry;
	  },
	  render: function() {
	    return <div className="voting">
	      {this.getPair().map(entry =>
	        <button key={entry}
	                disabled={this.isDisabled()}
	                onClick={() => this.props.vote(entry)}>
	          <h1>{entry}</h1>
	          {this.hasVotedFor(entry) ?
	            <div className="label">Voted</div> :
	            null}
	        </button>
	      )}
	    </div>;
	  }
	});

最后我们只需要在`Voting`组件做一下判断即可：

	//src/components/Voting.jsx

	import React from 'react';
	import Winner from './Winner';
	import Vote from './Vote';

	export default React.createClass({
	  render: function() {
	    return <div>
	      {this.props.winner ?
	        <Winner ref="winner" winner={this.props.winner} /> :
	        <Vote {...this.props} />}
	    </div>;
	  }
	});

注意这里我们为胜利组件添加了[ref](https://facebook.github.io/react/docs/more-about-refs.html)，这是因为我们将在单元测试中利用它获取DOM节点。

这就是我们的纯组件！注意目前我们还没有实现任何逻辑：我们并没有定义按钮的点击操作。组件只是用来渲染UI，其它
什么都不需要做。后面当我们将UI与Redux Store结合时才会涉及到应用逻辑。

继续下一步之前我们要为刚才新增的特性写更多的单元测试代码。首先，`hasVoted`属性将会使按钮改变状态：

	//test/components/Voting_spec.jsx

	it('disables buttons when user has voted', () => {
	  const component = renderIntoDocument(
	    <Voting pair={["Trainspotting", "28 Days Later"]}
	            hasVoted="Trainspotting" />
	  );
	  const buttons = scryRenderedDOMComponentsWithTag(component, 'button');

	  expect(buttons.length).to.equal(2);
	  expect(buttons[0].getDOMNode().hasAttribute('disabled')).to.equal(true);
	  expect(buttons[1].getDOMNode().hasAttribute('disabled')).to.equal(true);
	});

被`hasVoted`匹配的按钮将显示`Voted`标签：

	//test/components/Voting_spec.jsx

	it('adds label to the voted entry', () => {
	  const component = renderIntoDocument(
	    <Voting pair={["Trainspotting", "28 Days Later"]}
	            hasVoted="Trainspotting" />
	  );
	  const buttons = scryRenderedDOMComponentsWithTag(component, 'button');

	  expect(buttons[0].getDOMNode().textContent).to.contain('Voted');
	});

当获胜者产生，界面将不存在按钮，取而代替的是胜利者元素：

	//test/components/Voting_spec.jsx

	it('renders just the winner when there is one', () => {
	  const component = renderIntoDocument(
	    <Voting winner="Trainspotting" />
	  );
	  const buttons = scryRenderedDOMComponentsWithTag(component, 'button');
	  expect(buttons.length).to.equal(0);

	  const winner = React.findDOMNode(component.refs.winner);
	  expect(winner).to.be.ok;
	  expect(winner.textContent).to.contain('Trainspotting');
	});

### 5.5 不可变数据和纯粹渲染

我们之前已经讨论了许多关于不可变数据的红利，但是，当它和react结合时还会有一个非常屌的好处：
如果我们创建纯react组件并传递给它不可变数据作为属性参数，我们将会让react在组件渲染检测中得到最大性能。

这是靠react提供的[PureRenderMixin](https://facebook.github.io/react/docs/pure-render-mixin.html)实现的。
当该mixin添加到组件中后，组件的更新检查逻辑将会被改变，由深比对改为高性能的浅比对。

我们之所以可以使用浅比对，就是因为我们使用的是不可变数据。如果一个组件的所有参数都是不可变数据，
那么将大大提高应用性能。

我们可以在单元测试里更清楚的看见差别，如果我们向纯组件中传入可变数组，当数组内部元素产生改变后，组件并不会
重新渲染：

	//test/components/Voting_spec.jsx

	it('renders as a pure component', () => {
	  const pair = ['Trainspotting', '28 Days Later'];
	  const component = renderIntoDocument(
	    <Voting pair={pair} />
	  );

	  let firstButton = scryRenderedDOMComponentsWithTag(component, 'button')[0];
	  expect(firstButton.getDOMNode().textContent).to.equal('Trainspotting');

	  pair[0] = 'Sunshine';
	  component.setProps({pair: pair});
	  firstButton = scryRenderedDOMComponentsWithTag(component, 'button')[0];
	  expect(firstButton.getDOMNode().textContent).to.equal('Trainspotting');
	});

如果我们使用不可变数据，则完全没有问题：

	//test/components/Voting_spec.jsx

	import React from 'react/addons';
	import {List} from 'immutable';
	import Voting from '../../src/components/Voting';
	import {expect} from 'chai';

	const {renderIntoDocument, scryRenderedDOMComponentsWithTag, Simulate}
	  = React.addons.TestUtils;

	describe('Voting', () => {

	  // ...

	  it('does update DOM when prop changes', () => {
	    const pair = List.of('Trainspotting', '28 Days Later');
	    const component = renderIntoDocument(
	      <Voting pair={pair} />
	    );

	    let firstButton = scryRenderedDOMComponentsWithTag(component, 'button')[0];
	    expect(firstButton.getDOMNode().textContent).to.equal('Trainspotting');

	    const newPair = pair.set(0, 'Sunshine');
	    component.setProps({pair: newPair});
	    firstButton = scryRenderedDOMComponentsWithTag(component, 'button')[0];
	    expect(firstButton.getDOMNode().textContent).to.equal('Sunshine');
	  });
	});

如果你跑上面的两个测试，你将会看到非预期的结果：因为实际上UI在两种场景下都更新了。那是因为现在组件
依然使用的是深比对，这正是我们使用不可变数据想极力避免的。

下面我们在组件中引入mixin，你就会拿到期望的结果了：

	//src/components/Voting.jsx

	import React from 'react/addons';
	import Winner from './Winner';
	import Vote from './Vote';

	export default React.createClass({
	  mixins: [React.addons.PureRenderMixin],
	  // ...
	});



	//src/components/Vote.jsx

	import React from 'react/addons';

	export default React.createClass({
	  mixins: [React.addons.PureRenderMixin],
	  // ...
	});



	//src/components/Winner.jsx

	import React from 'react/addons';

	export default React.createClass({
	  mixins: [React.addons.PureRenderMixin],
	  // ...
	});

### 5.6 投票结果页面和路由实现

投票页面已经搞定了，让我们开始实现投票结果页面吧。

投票结果页面依然会显示两个条目，并且显示它们各自的票数。此外屏幕下方还会有一个按钮，供用户切换到下一轮投票。

现在我们根据什么来确定显示哪个界面呢？使用URL是个不错的主意：我们可以设置根路径`#/`去显示投票页面，
使用`#/results`来显示投票结果页面。

我们使用[react-router](http://rackt.github.io/react-router/)可以很容易实现这个需求。让我们加入项目：

	npm install --save react-router

我们这里使用的react-router的0.13版本，它的1.0版本官方还没有发布，如果你打算使用其1.0RC版，那么下面的代码
你可能需要做一些修改，可以看[router文档](https://github.com/rackt/react-router)。

我们现在可以来配置一下路由路径，Router提供了一个`Route`组件用来让我们定义路由信息，同时也提供了`DefaultRoute`
组件来让我们定义默认路由：

	//src/index.jsx

	import React from 'react';
	import {Route, DefaultRoute} from 'react-router';
	import App from './components/App';
	import Voting from './components/Voting';

	const pair = ['Trainspotting', '28 Days Later'];

	const routes = <Route handler={App}>
	  <DefaultRoute handler={Voting} />
	</Route>;

	React.render(
	  <Voting pair={pair} />,
	  document.getElementById('app')
	);

我们定义了一个默认的路由指向我们的`Voting`组件。我们需要定义个`App`组件来用于Route使用。

根路由的作用就是为应用指定一个根组件：通常该组件充当所有子页面的模板。让我们来看看`App`的细节：

	//src/components/App.jsx

	import React from 'react';
	import {RouteHandler} from 'react-router';
	import {List} from 'immutable';

	const pair = List.of('Trainspotting', '28 Days Later');

	export default React.createClass({
	  render: function() {
	    return <RouteHandler pair={pair} />
	  }
	});

这个组件除了渲染了一个`RouteHandler`组件并没有做别的，这个组件同样是react-router提供的，它的作用就是
每当路由匹配了某个定义的页面后将对应的页面组件插入到这个位置。目前我们只定义了一个默认路由指向`Voting`，
所以目前我们的组件总是会显示`Voting`界面。

注意，我们将我们硬编码的投票数据从`index.jsx`移到了`App.jsx`，当你给`RouteHandler`传递了属性值时，
这些参数将会传给当前路由对应的组件。

现在我们可以更新`index.jsx`：

	//src/index.jsx

	import React from 'react';
	import Router, {Route, DefaultRoute} from 'react-router';
	import App from './components/App';
	import Voting from './components/Voting';

	const routes = <Route handler={App}>
	  <DefaultRoute handler={Voting} />
	</Route>;

	Router.run(routes, (Root) => {
	  React.render(
	    <Root />,
	    document.getElementById('app')
	  );
	});

`run`方法会根据当前浏览器的路径去查找定义的router来决定渲染哪个组件。一旦确定了对应的组件，它将会被
当作指定的`Root`传给`run`的回调函数，在回调中我们将使用`React.render`将其插入DOM中。

目前为止我们已经基于React router实现了之前的内容，我们现在可以很容易添加更多新的路由到应用。让我们
把投票结果页面添加进去吧：

	//src/index.jsx

	import React from 'react';
	import Router, {Route, DefaultRoute} from 'react-router';
	import App from './components/App';
	import Voting from './components/Voting';
	import Results from './components/Results';

	const routes = <Route handler={App}>
	  <Route path="/results" handler={Results} />
	  <DefaultRoute handler={Voting} />
	</Route>;

	Router.run(routes, (Root) => {
	  React.render(
	    <Root />,
	    document.getElementById('app')
	  );
	});

这里我们用使用`<Route>`组件定义了一个名为`/results`的路径，并绑定`Results`组件。

让我们简单的实现一下这个`Results`组件，这样我们就可以看一下路由是如何工作的了：

	//src/components/Results.jsx

	import React from 'react/addons';

	export default React.createClass({
	  mixins: [React.addons.PureRenderMixin],
	  render: function() {
	    return <div>Hello from results!</div>
	  }
	});

如果你在浏览器中输入[http://localhost:8080/#/results](http://localhost:8080/#/results)，你将会看到该结果组件。
而其它路径都对应这投票页面，你也可以使用浏览器的前后按钮来切换这两个界面。

接下来我们来实际实现一下结果组件：

	//src/components/Results.jsx

	import React from 'react/addons';

	export default React.createClass({
	  mixins: [React.addons.PureRenderMixin],
	  getPair: function() {
	    return this.props.pair || [];
	  },
	  render: function() {
	    return <div className="results">
	      {this.getPair().map(entry =>
	        <div key={entry} className="entry">
	          <h1>{entry}</h1>
	        </div>
	      )}
	    </div>;
	  }
	});

结果界面除了显示投票项外，还应该显示它们对应的得票数，让我们先硬编码一下：

	//src/components/App.jsx

	import React from 'react/addons';
	import {RouteHandler} from 'react-router';
	import {List, Map} from 'immutable';

	const pair = List.of('Trainspotting', '28 Days Later');
	const tally = Map({'Trainspotting': 5, '28 Days Later': 4});

	export default React.createClass({
	  render: function() {
	    return <RouteHandler pair={pair}
	                         tally={tally} />
	  }
	});

现在，我们再来修改一下结果组件：

	//src/components/Results.jsx

	import React from 'react/addons';

	export default React.createClass({
	  mixins: [React.addons.PureRenderMixin],
	  getPair: function() {
	    return this.props.pair || [];
	  },
	  getVotes: function(entry) {
	    if (this.props.tally && this.props.tally.has(entry)) {
	      return this.props.tally.get(entry);
	    }
	    return 0;
	  },
	  render: function() {
	    return <div className="results">
	      {this.getPair().map(entry =>
	        <div key={entry} className="entry">
	          <h1>{entry}</h1>
	          <div className="voteCount">
	            {this.getVotes(entry)}
	          </div>
	        </div>
	      )}
	    </div>;
	  }
	});

现在我们来针对目前的界面功能编写测试代码，以防止未来我们破坏这些功能。

我们期望组件为每个选项都渲染一个div，并在其中显示选项的名称和票数。如果对应的选项没有票数，则默认显示0：

	//test/components/Results_spec.jsx

	import React from 'react/addons';
	import {List, Map} from 'immutable';
	import Results from '../../src/components/Results';
	import {expect} from 'chai';

	const {renderIntoDocument, scryRenderedDOMComponentsWithClass}
	  = React.addons.TestUtils;

	describe('Results', () => {

	  it('renders entries with vote counts or zero', () => {
	    const pair = List.of('Trainspotting', '28 Days Later');
	    const tally = Map({'Trainspotting': 5});
	    const component = renderIntoDocument(
	      <Results pair={pair} tally={tally} />
	    );
	    const entries = scryRenderedDOMComponentsWithClass(component, 'entry');
	    const [train, days] = entries.map(e => e.getDOMNode().textContent);

	    expect(entries.length).to.equal(2);
	    expect(train).to.contain('Trainspotting');
	    expect(train).to.contain('5');
	    expect(days).to.contain('28 Days Later');
	    expect(days).to.contain('0');
	  });
	});

接下来，我们看一下"Next"按钮，它允许用户切换到下一轮投票。

我们的组件应该包含一个回调函数属性参数，当组件中的"Next"按钮被点击后，该回调函数将会被调用。我们来写一下
这个操作的测试代码：

	//test/components/Results_spec.jsx

	import React from 'react/addons';
	import {List, Map} from 'immutable';
	import Results from '../../src/components/Results';
	import {expect} from 'chai';

	const {renderIntoDocument, scryRenderedDOMComponentsWithClass, Simulate}
	  = React.addons.TestUtils;

	describe('Results', () => {

	  // ...

	  it('invokes the next callback when next button is clicked', () => {
	    let nextInvoked = false;
	    const next = () => nextInvoked = true;

	    const pair = List.of('Trainspotting', '28 Days Later');
	    const component = renderIntoDocument(
	      <Results pair={pair}
	               tally={Map()}
	               next={next}/>
	    );
	    Simulate.click(React.findDOMNode(component.refs.next));

	    expect(nextInvoked).to.equal(true);
	  });
	});

写法和之前的投票按钮很类似吧。接下来让我们更新一下结果组件：

	//src/components/Results.jsx

	import React from 'react/addons';

	export default React.createClass({
	  mixins: [React.addons.PureRenderMixin],
	  getPair: function() {
	    return this.props.pair || [];
	  },
	  getVotes: function(entry) {
	    if (this.props.tally && this.props.tally.has(entry)) {
	      return this.props.tally.get(entry);
	    }
	    return 0;
	  },
	  render: function() {
	    return <div className="results">
	      <div className="tally">
	        {this.getPair().map(entry =>
	          <div key={entry} className="entry">
	            <h1>{entry}</h1>
	            <div class="voteCount">
	              {this.getVotes(entry)}
	            </div>
	          </div>
	        )}
	      </div>
	      <div className="management">
	        <button ref="next"
	                className="next"
	                onClick={this.props.next}>
	          Next
	        </button>
	      </div>
	    </div>;
	  }
	});

最终投票结束，结果页面和投票页面一样，都要显示胜利者：

	//test/components/Results_spec.jsx

	it('renders the winner when there is one', () => {
	  const component = renderIntoDocument(
	    <Results winner="Trainspotting"
	             pair={["Trainspotting", "28 Days Later"]}
	             tally={Map()} />
	  );
	  const winner = React.findDOMNode(component.refs.winner);
	  expect(winner).to.be.ok;
	  expect(winner.textContent).to.contain('Trainspotting');
	});

我们可以想在投票界面中那样简单的实现一下上面的逻辑：

	//src/components/Results.jsx

	import React from 'react/addons';
	import Winner from './Winner';

	export default React.createClass({
	  mixins: [React.addons.PureRenderMixin],
	  getPair: function() {
	    return this.props.pair || [];
	  },
	  getVotes: function(entry) {
	    if (this.props.tally && this.props.tally.has(entry)) {
	      return this.props.tally.get(entry);
	    }
	    return 0;
	  },
	  render: function() {
	    return this.props.winner ?
	      <Winner ref="winner" winner={this.props.winner} /> :
	      <div className="results">
	        <div className="tally">
	          {this.getPair().map(entry =>
	            <div key={entry} className="entry">
	              <h1>{entry}</h1>
	              <div className="voteCount">
	                {this.getVotes(entry)}
	              </div>
	            </div>
	          )}
	        </div>
	        <div className="management">
	          <button ref="next"
	                   className="next"
	                   onClick={this.props.next}>
	            Next
	          </button>
	        </div>
	      </div>;
	  }
	});

到目前为止，我们已经实现了应用的UI，虽然现在它们并没有和真实数据和操作整合起来。这很不错不是么？
我们只需要一些占位符数据就可以完成界面的开发，这让我们在这个阶段更专注于UI。

接下来我们将会使用Redux Store来将真实数据整合到我们的界面中。

### 5.7 初识客户端的Redux Store

Redux将会充当我们UI界面的状态容器，我们已经在服务端用过Redux，之前说的很多内容在这里也受用。
现在我们已经准备好要在React应用中使用Redux了，这也是Redux更常见的使用场景。

和在服务端一样，我们先来思考一下应用的状态。客户端的状态和服务端会非常的类似。

我们有两个界面，并在其中需要显示成对的用于投票的条目：

![](http://teropa.info/images/vote_client_pair.png)

此外，结果页面需要显示票数：

![](http://teropa.info/images/vote_client_tally.png)

投票组件还需要记录当前用户已经投票过的选项：

![](http://teropa.info/images/vote_client_hasvoted.png)

结果组件还需要记录胜利者：

![](http://teropa.info/images/vote_server_tree_winner.png)

注意这里除了`hasVoted`外，其它都映射着服务端状态的子集。

接下来我们来思考一下应用的核心逻辑，actions和reducers应该是什么样的。

我们先来想想能够导致应用状态改变的操作都有那些？状态改变的来源之一是用户行为。我们的UI中存在两种
可能的用户操作行为：

- 用户在投票页面点击某个投票按钮；
- 用户点击下一步按钮。

另外，我们知道我们的服务端会将应用当前状态发送给客户端，我们将编写代码来接受状态数据，这也是导致状态
改变的来源之一。

我们可以从服务端状态更新开始，之前我们在服务端设置发送了一个`state`事件。该事件将携带我们之前设计的客户端
状态树的状态数据。我们的客户端reducer将通过一个action来将服务器端的状态数据合并到客户端状态树中，
这个action如下：

	{
	  type: 'SET_STATE',
	  state: {
	    vote: {...}
	  }
	}

让我们先写一下reducer测试代码，它应该接受上面定义的那种action，并合并数据到客户端的当前状态中：

	//test/reducer_spec.js

	import {List, Map, fromJS} from 'immutable';
	import {expect} from 'chai';

	import reducer from '../src/reducer';

	describe('reducer', () => {

	  it('handles SET_STATE', () => {
	    const initialState = Map();
	    const action = {
	      type: 'SET_STATE',
	      state: Map({
	        vote: Map({
	          pair: List.of('Trainspotting', '28 Days Later'),
	          tally: Map({Trainspotting: 1})
	        })
	      })
	    };
	    const nextState = reducer(initialState, action);

	    expect(nextState).to.equal(fromJS({
	      vote: {
	        pair: ['Trainspotting', '28 Days Later'],
	        tally: {Trainspotting: 1}
	      }
	    }));
	  });
	});

这个renducers接受一个来自socket发送的原始的js数据结构，这里注意不是不可变数据类型哦。我们需要在返回前将其
转换成不可变数据类型：

	//test/reducer_spec.js

	it('handles SET_STATE with plain JS payload', () => {
	  const initialState = Map();
	  const action = {
	    type: 'SET_STATE',
	    state: {
	      vote: {
	        pair: ['Trainspotting', '28 Days Later'],
	        tally: {Trainspotting: 1}
	      }
	    }
	  };
	  const nextState = reducer(initialState, action);

	  expect(nextState).to.equal(fromJS({
	    vote: {
	      pair: ['Trainspotting', '28 Days Later'],
	      tally: {Trainspotting: 1}
	    }
	  }));
	});

reducer同样应该可以正确的处理`undefined`初始化状态：

	//test/reducer_spec.js

	it('handles SET_STATE without initial state', () => {
	  const action = {
	    type: 'SET_STATE',
	    state: {
	      vote: {
	        pair: ['Trainspotting', '28 Days Later'],
	        tally: {Trainspotting: 1}
	      }
	    }
	  };
	  const nextState = reducer(undefined, action);

	  expect(nextState).to.equal(fromJS({
	    vote: {
	      pair: ['Trainspotting', '28 Days Later'],
	      tally: {Trainspotting: 1}
	    }
	  }));
	});

现在我们来看一下如何实现满足上面测试条件的reducer：

	//src/reducer.js

	import {Map} from 'immutable';

	export default function(state = Map(), action) {

	  return state;
	}

reducer需要处理`SET_STATE`动作。在这个动作的处理中，我们应该将传入的状态数据和现有的进行合并，
使用Map提供的[merge](https://facebook.github.io/immutable-js/docs/#/Map/merge)将很容易来实现这个操作：

	//src/reducer.js

	import {Map} from 'immutable';

	function setState(state, newState) {
	  return state.merge(newState);
	}

	export default function(state = Map(), action) {
	  switch (action.type) {
	  case 'SET_STATE':
	    return setState(state, action.state);
	  }
	  return state;
	}

注意这里我们并没有单独写一个核心模块，而是直接在reducer中添加了个简单的`setState`函数来做业务逻辑。
这是因为现在这个逻辑还很简单～

关于改变用户状态的那两个用户交互：投票和下一步，它们都需要和服务端进行通信，我们一会再说。我们现在先把
redux添加到项目中：

	npm install --save redux

`index.jsx`入口文件是一个初始化Store的好地方，让我们暂时先使用硬编码的数据来做：

	//src/index.jsx

	import React from 'react';
	import Router, {Route, DefaultRoute} from 'react-router';
	import {createStore} from 'redux';
	import reducer from './reducer';
	import App from './components/App';
	import Voting from './components/Voting';
	import Results from './components/Results';

	const store = createStore(reducer);
	store.dispatch({
	  type: 'SET_STATE',
	  state: {
	    vote: {
	      pair: ['Sunshine', '28 Days Later'],
	      tally: {Sunshine: 2}
	    }
	  }
	});

	const routes = <Route handler={App}>
	  <Route path="/results" handler={Results} />
	  <DefaultRoute handler={Voting} />
	</Route>;

	Router.run(routes, (Root) => {
	  React.render(
	    <Root />,
	    document.getElementById('app')
	  );
	});

那么，我们如何在react组件中从Store中获取数据呢？

### 5.8 让React从Redux中获取数据

我们已经创建了一个使用不可变数据类型保存应用状态的Redux Store。我们还拥有接受不可变数据为参数的
无状态的纯React组件。如果我们能使这些组件从Store中获取最新的状态数据，那真是极好的。当状态变化时，
React会重新渲染组件，pure render mixin可以使得我们的UI避免不必要的重复渲染。

相比我们自己手动实现同步代码，我们更推荐使用[react-redux][https://github.com/rackt/react-redux]包来做：

	npm install --save react-redux

这个库主要做的是：

1. 映射Store的状态到组件的输入props中；
2. 映射actions到组件的回调props中。

为了让它可以正常工作，我们需要将顶层的应用组件嵌套在react-redux的[Provider](https://github.com/rackt/react-redux#provider-store)组件中。
这将把Redux Store和我们的状态树连接起来。

我们将让Provider包含路由的根组件，这样会使得Provider成为整个应用组件的根节点：

	//src/index.jsx

	import React from 'react';
	import Router, {Route, DefaultRoute} from 'react-router';
	import {createStore} from 'redux';
	import {Provider} from 'react-redux';
	import reducer from './reducer';
	import App from './components/App';
	import {VotingContainer} from './components/Voting';
	import Results from './components/Results';

	const store = createStore(reducer);
	store.dispatch({
	  type: 'SET_STATE',
	  state: {
	    vote: {
	      pair: ['Sunshine', '28 Days Later'],
	      tally: {Sunshine: 2}
	    }
	  }
	});

	const routes = <Route handler={App}>
	  <Route path="/results" handler={Results} />
	  <DefaultRoute handler={VotingContainer} />
	</Route>;

	Router.run(routes, (Root) => {
	  React.render(
	    <Provider store={store}>
	      {() => <Root />}
	    </Provider>,
	    document.getElementById('app')
	  );
	});

接下来我们要考虑一下，我们的那些组件需要绑定到Store上。我们一共有5个组件，可以分成三类：

- 根组件`App`不需要绑定任何数据；
- `Vote`和`Winner`组件只使用父组件传递来的数据，所以它们也不需要绑定；
- 剩下的组件（`Voting`和`Results`）目前都是使用的硬编码数据，我们现在需要将其绑定到Store上。

让我们从`Voting`组件开始。使用react-redux我们得到一个叫[connect](https://github.com/rackt/react-redux#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options)的函数：

	connect(mapStateToProps)(SomeComponent);

该函数的作用就是将Redux Store中的状态数据映射到props对象中。这个props对象将会用于连接到的组件中。
在我们的`Voting`场景中，我们需要从状态中拿到`pair`和`winner`值：

	//src/components/Voting.jsx

	import React from 'react/addons';
	import {connect} from 'react-redux';
	import Winner from './Winner';
	import Vote from './Vote';

	const Voting = React.createClass({
	  mixins: [React.addons.PureRenderMixin],
	  render: function() {
	    return <div>
	      {this.props.winner ?
	        <Winner ref="winner" winner={this.props.winner} /> :
	        <Vote {...this.props} />}
	    </div>;
	  }
	});

	function mapStateToProps(state) {
	  return {
	    pair: state.getIn(['vote', 'pair']),
	    winner: state.get('winner')
	  };
	}

	connect(mapStateToProps)(Voting);

	export default Voting;

在上面的代码中，`connect`函数并没有修改`Voting`组件本身，`Voting`组件依然保持这纯粹性。而`connect`
返回的是一个`Voting`组件的连接版，我们称之为`VotingContainer`：

	//src/components/Voting.jsx

	import React from 'react/addons';
	import {connect} from 'react-redux';
	import Winner from './Winner';
	import Vote from './Vote';

	export const Voting = React.createClass({
	  mixins: [React.addons.PureRenderMixin],
	  render: function() {
	    return <div>
	      {this.props.winner ?
	        <Winner ref="winner" winner={this.props.winner} /> :
	        <Vote {...this.props} />}
	    </div>;
	  }
	});

	function mapStateToProps(state) {
	  return {
	    pair: state.getIn(['vote', 'pair']),
	    winner: state.get('winner')
	  };
	}

	export const VotingContainer = connect(mapStateToProps)(Voting);

这样，这个模块现在导出两个组件：一个纯`Voting`组件，一个连接后的`VotingContainer`版本。
react-redux官方称前者为“蠢”组件，后者则称为"智能"组件。我更倾向于用“pure”和“connected”来描述它们。
怎么称呼随你便，主要是明白它们之间的差别：

- 纯组件完全靠给它传入的props来工作，这非常类似一个纯函数；
- 连接组件则封装了纯组件和一些逻辑用来与Redux Store协同工作，这些特性是redux-react提供的。

我们得更新一下路由表，改用`VotingContainer`。一旦修改完毕，我们的投票界面将会使用来自Redux Store的数据：

	//src/index.jsx

	import React from 'react';
	import Router, {Route, DefaultRoute} from 'react-router';
	import {createStore} from 'redux';
	import {Provider} from 'react-redux';
	import reducer from './reducer';
	import App from './components/App';
	import {VotingContainer} from './components/Voting';
	import Results from './components/Results';

	const store = createStore(reducer);
	store.dispatch({
	  type: 'SET_STATE',
	  state: {
	    vote: {
	      pair: ['Sunshine', '28 Days Later'],
	      tally: {Sunshine: 2}
	    }
	  }
	});

	const routes = <Route handler={App}>
	  <Route path="/results" handler={Results} />
	  <DefaultRoute handler={VotingContainer} />
	</Route>;

	Router.run(routes, (Root) => {
	  React.render(
	    <Provider store={store}>
	      {() => <Root />}
	    </Provider>,
	    document.getElementById('app')
	  );
	});

而在对应的测试代码中，我们则需要使用纯`Voting`组件定义：

	//test/components/Voting_spec.jsx

	import React from 'react/addons';
	import {List} from 'immutable';
	import {Voting} from '../../src/components/Voting';
	import {expect} from 'chai';

其它地方不需要修改了。

现在我们来如法炮制投票结果页面：

	//src/components/Results.jsx

	import React from 'react/addons';
	import {connect} from 'react-redux';
	import Winner from './Winner';

	export const Results = React.createClass({
	  mixins: [React.addons.PureRenderMixin],
	  getPair: function() {
	    return this.props.pair || [];
	  },
	  getVotes: function(entry) {
	    if (this.props.tally && this.props.tally.has(entry)) {
	      return this.props.tally.get(entry);
	    }
	    return 0;
	  },
	  render: function() {
	    return this.props.winner ?
	      <Winner ref="winner" winner={this.props.winner} /> :
	      <div className="results">
	        <div className="tally">
	          {this.getPair().map(entry =>
	            <div key={entry} className="entry">
	              <h1>{entry}</h1>
	              <div className="voteCount">
	                {this.getVotes(entry)}
	              </div>
	            </div>
	          )}
	        </div>
	        <div className="management">
	          <button ref="next"
	                   className="next"
	                   onClick={this.props.next}>
	            Next
	          </button>
	      </div>
	      </div>;
	  }
	});

	function mapStateToProps(state) {
	  return {
	    pair: state.getIn(['vote', 'pair']),
	    tally: state.getIn(['vote', 'tally']),
	    winner: state.get('winner')
	  }
	}

	export const ResultsContainer = connect(mapStateToProps)(Results);

同样我们需要修改`index.jsx`来使用新的`ResultsContainer`：

	//src/index.jsx

	import React from 'react';
	import Router, {Route, DefaultRoute} from 'react-router';
	import {createStore} from 'redux';
	import {Provider} from 'react-redux';
	import reducer from './reducer';
	import App from './components/App';
	import {VotingContainer} from './components/Voting';
	import {ResultsContainer} from './components/Results';

	const store = createStore(reducer);
	store.dispatch({
	  type: 'SET_STATE',
	  state: {
	    vote: {
	      pair: ['Sunshine', '28 Days Later'],
	      tally: {Sunshine: 2}
	    }
	  }
	});

	const routes = <Route handler={App}>
	  <Route path="/results" handler={ResultsContainer} />
	  <DefaultRoute handler={VotingContainer} />
	</Route>;

	Router.run(routes, (Root) => {
	  React.render(
	    <Provider store={store}>
	      {() => <Root />}
	    </Provider>,
	    document.getElementById('app')
	  );
	});

不要忘记修改测试代码啊：

	//test/components/Results_spec.jsx

	import React from 'react/addons';
	import {List, Map} from 'immutable';
	import {Results} from '../../src/components/Results';
	import {expect} from 'chai';

现在你已经知道如何让纯react组件与Redux Store整合了。

对于一些只有一个根组件且没有路由的小应用，直接连接根组件就足够了。根组件会将状态数据传递给它的子组件。
而对于那些使用路由，就像我们的场景，连接每一个路由指向的处理函数是个好主意。但是分别为每个组件编写连接代码并
不适合所有的软件场景。我觉得保持组件props尽可能清晰明了是个非常好的习惯，因为它可以让你很容易清楚组件需要哪些数据，
你就可以更容易管理那些连接代码。

现在让我们开始把Redux数据对接到UI里，我们再也不需要那些`App.jsx`中手写的硬编码数据了，这样我们的`App.jsx`将会变得简单：

	//src/components/App.jsx

	import React from 'react';
	import {RouteHandler} from 'react-router';

	export default React.createClass({
	  render: function() {
	    return <RouteHandler />
	  }
	});


### 5.9 设置socket.io客户端

现在我们已经创建好了客户端的Redux应用，我们接下来将讨论如何让其与我们之前开发的服务端应用进行对接。

服务端已经准备好接受socket连接，并为其进行投票数据的发送。而我们的客户端也已经可以使用Redux Store很方便的
接受数据了。我们剩下的工作就是把它们连接起来。

我们需要使用socket.io从浏览器向服务端创建一个连接，我们可以使用[socket.io-client库](http://socket.io/docs/client-api/)来完成
这个目的：

	npm install --save socket.io-client

这个库赋予了我们连接Socket.io服务端的能力，让我们连接之前写好的服务端，端口号8090（注意使用和后端匹配的端口）：

	//src/index.jsx

	import React from 'react';
	import Router, {Route, DefaultRoute} from 'react-router';
	import {createStore} from 'redux';
	import {Provider} from 'react-redux';
	import io from 'socket.io-client';
	import reducer from './reducer';
	import App from './components/App';
	import {VotingContainer} from './components/Voting';
	import {ResultsContainer} from './components/Results';

	const store = createStore(reducer);
	store.dispatch({
	  type: 'SET_STATE',
	  state: {
	    vote: {
	      pair: ['Sunshine', '28 Days Later'],
	      tally: {Sunshine: 2}
	    }
	  }
	});

	const socket = io(`${location.protocol}//${location.hostname}:8090`);

	const routes = <Route handler={App}>
	  <Route path="/results" handler={ResultsContainer} />
	  <DefaultRoute handler={VotingContainer} />
	</Route>;

	Router.run(routes, (Root) => {
	  React.render(
	    <Provider store={store}>
	      {() => <Root />}
	    </Provider>,
	    document.getElementById('app')
	  );
	});

你必须先确保你的服务端已经开启了，然后在浏览器端访问客户端应用，并检查网络监控，你会发现创建了一个
WebSockets连接，并且开始传输Socket.io的心跳包了。

### 5.10 接受来自服务器端的actions

我们虽然已经创建了个socket.io连接，但我们并没有用它获取任何数据。每当我们连接到服务端或服务端发生
状态数据改变时，服务端会发送`state`事件给客户端。我们只需要监听对应的事件即可，我们在接受到事件通知后
只需要简单的对我们的Store指派`SET_STATE`action即可：

	//src/index.jsx

	import React from 'react';
	import Router, {Route, DefaultRoute} from 'react-router';
	import {createStore} from 'redux';
	import {Provider} from 'react-redux';
	import io from 'socket.io-client';
	import reducer from './reducer';
	import App from './components/App';
	import {VotingContainer} from './components/Voting';
	import {ResultsContainer} from './components/Results';

	const store = createStore(reducer);

	const socket = io(`${location.protocol}//${location.hostname}:8090`);
	socket.on('state', state =>
	  store.dispatch({type: 'SET_STATE', state})
	);

	const routes = <Route handler={App}>
	  <Route path="/results" handler={ResultsContainer} />
	  <DefaultRoute handler={VotingContainer} />
	</Route>;

	Router.run(routes, (Root) => {
	  React.render(
	    <Provider store={store}>
	      {() => <Root />}
	    </Provider>,
	    document.getElementById('app')
	  );
	});

注意我们移除了`SET_STATE`的硬编码，我们现在已经不需要伪造数据了。

审视我们的界面，不管是投票还是结果页面，它们都会显示服务端提供的第一对选项。服务端和客户端已经连接上了！

### 5.11 从react组件中指派actions

我们已经知道如何从Redux Store获取数据到UI中，现在来看看如何从UI中提交数据用于actions。

思考这个问题的最佳场景是投票界面上的投票按钮。之前在写相关界面时，我们假设`Voting`组件接受一个回调函数props。
当用户点击某个按钮时组件将会调用这个回调函数。但我们目前并没有实现这个回调函数，除了在测试代码中。

当用户投票后应该做什么？投票结果应该发送给服务端，这部分我们稍后再说，客户端也需要执行一些逻辑：
组件的`hasVoted`值应该被设置，这样用户才不会反复对同一对选项投票。

这是我们要创建的第二个客户端Redux Action，我们称之为`VOTE`：

	//test/reducer_spec.js

	it('handles VOTE by setting hasVoted', () => {
	  const state = fromJS({
	    vote: {
	      pair: ['Trainspotting', '28 Days Later'],
	      tally: {Trainspotting: 1}
	    }
	  });
	  const action = {type: 'VOTE', entry: 'Trainspotting'};
	  const nextState = reducer(state, action);

	  expect(nextState).to.equal(fromJS({
	    vote: {
	      pair: ['Trainspotting', '28 Days Later'],
	      tally: {Trainspotting: 1}
	    },
	    hasVoted: 'Trainspotting'
	  }));
	});

为了更严谨，我们应该考虑一种情况：不管什么原因，当`VOTE`action传递了一个不存在的选项时我们的应用该怎么做：

	//test/reducer_spec.js

	it('does not set hasVoted for VOTE on invalid entry', () => {
	  const state = fromJS({
	    vote: {
	      pair: ['Trainspotting', '28 Days Later'],
	      tally: {Trainspotting: 1}
	    }
	  });
	  const action = {type: 'VOTE', entry: 'Sunshine'};
	  const nextState = reducer(state, action);

	  expect(nextState).to.equal(fromJS({
	    vote: {
	      pair: ['Trainspotting', '28 Days Later'],
	      tally: {Trainspotting: 1}
	    }
	  }));
	});

下面来看看我们的reducer如何实现的：

	//src/reducer.js

	import {Map} from 'immutable';

	function setState(state, newState) {
	  return state.merge(newState);
	}

	function vote(state, entry) {
	  const currentPair = state.getIn(['vote', 'pair']);
	  if (currentPair && currentPair.includes(entry)) {
	    return state.set('hasVoted', entry);
	  } else {
	    return state;
	  }
	}

	export default function(state = Map(), action) {
	  switch (action.type) {
	  case 'SET_STATE':
	    return setState(state, action.state);
	  case 'VOTE':
	    return vote(state, action.entry);
	  }
	  return state;
	}

`hasVoted`并不会一直保存在状态数据中，每当开始一轮新的投票时，我们应该在`SET_STATE`action的处理逻辑中
检查是否用户是否已经投票，如果还没，我们应该删除掉`hasVoted`：

	//test/reducer_spec.js

	it('removes hasVoted on SET_STATE if pair changes', () => {
	  const initialState = fromJS({
	    vote: {
	      pair: ['Trainspotting', '28 Days Later'],
	      tally: {Trainspotting: 1}
	    },
	    hasVoted: 'Trainspotting'
	  });
	  const action = {
	    type: 'SET_STATE',
	    state: {
	      vote: {
	        pair: ['Sunshine', 'Slumdog Millionaire']
	      }
	    }
	  };
	  const nextState = reducer(initialState, action);

	  expect(nextState).to.equal(fromJS({
	    vote: {
	      pair: ['Sunshine', 'Slumdog Millionaire']
	    }
	  }));
	});

根据需要，我们新增一个`resetVote`函数来处理`SET_STATE`动作：

	//src/reducer.js

	import {List, Map} from 'immutable';

	function setState(state, newState) {
	  return state.merge(newState);
	}

	function vote(state, entry) {
	  const currentPair = state.getIn(['vote', 'pair']);
	  if (currentPair && currentPair.includes(entry)) {
	    return state.set('hasVoted', entry);
	  } else {
	    return state;
	  }
	}

	function resetVote(state) {
	  const hasVoted = state.get('hasVoted');
	  const currentPair = state.getIn(['vote', 'pair'], List());
	  if (hasVoted && !currentPair.includes(hasVoted)) {
	    return state.remove('hasVoted');
	  } else {
	    return state;
	  }
	}

	export default function(state = Map(), action) {
	  switch (action.type) {
	  case 'SET_STATE':
	    return resetVote(setState(state, action.state));
	  case 'VOTE':
	    return vote(state, action.entry);
	  }
	  return state;
	}

我们还需要在修改一下连接逻辑：

	//src/components/Voting.jsx

	function mapStateToProps(state) {
	  return {
	    pair: state.getIn(['vote', 'pair']),
	    hasVoted: state.get('hasVoted'),
	    winner: state.get('winner')
	  };
	}

现在我们依然需要为`Voting`提供一个`vote`回调函数，用来为Sotre指派我们新增的action。我们依然要尽力保证
`Voting`组件的纯粹性，不应该依赖任何actions或Redux。这些工作都应该在react-redux的`connect`中处理。

除了连接输入参数属性，react-redux还可以用来连接output actions。开始之前，我们先来介绍一下另一个Redux的
核心概念：Action creators。

如我们之前看到的，Redux actions通常就是一个简单的对象，它包含一个固有的`type`属性和其它内容。我们之前都是直接
利用js对象字面量来直接声明所需的actions。其实可以使用一个factory函数来更好的生成actions，如下：

	function vote(entry) {
	  return {type: 'VOTE', entry};
	}

这类函数就被称为action creators。它们就是个纯函数，用来返回action对象，别的没啥好介绍得了。但是你也可以
在其中实现一些内部逻辑，而避免将每次生成action都重复编写它们。使用action creators可以更好的表达所有需要分发
的actions。

让我们新建一个用来声明客户端所需action的action creators文件：

	//src/action_creators.js

	export function setState(state) {
	  return {
	    type: 'SET_STATE',
	    state
	  };
	}

	export function vote(entry) {
	  return {
	    type: 'VOTE',
	    entry
	  };
	}

我们当然也可以为action creators编写测试代码，但由于我们的代码逻辑太简单了，我就不再写测试了。

现在我们可以在`index.jsx`中使用我们刚新增的`setState`action creator了：

	//src/index.jsx

	import React from 'react';
	import Router, {Route, DefaultRoute} from 'react-router';
	import {createStore} from 'redux';
	import {Provider} from 'react-redux';
	import io from 'socket.io-client';
	import reducer from './reducer';
	import {setState} from './action_creators';
	import App from './components/App';
	import {VotingContainer} from './components/Voting';
	import {ResultsContainer} from './components/Results';

	const store = createStore(reducer);

	const socket = io(`${location.protocol}//${location.hostname}:8090`);
	socket.on('state', state =>
	  store.dispatch(setState(state))
	);

	const routes = <Route handler={App}>
	  <Route path="/results" handler={ResultsContainer} />
	  <DefaultRoute handler={VotingContainer} />
	</Route>;

	Router.run(routes, (Root) => {
	  React.render(
	    <Provider store={store}>
	      {() => <Root />}
	    </Provider>,
	    document.getElementById('app')
	  );
	});

使用action creators还有一个非常优雅的特点：在我们的场景里，我们有一个需要`vote`回调函数props的
`Vote`组件，我们同时拥有一个`vote`的action creator。它们的名字和函数签名完全一致（都接受一个用来表示
选中项的参数）。现在我们只需要将action creators作为react-redux的`connect`函数的第二个参数，即可完成
自动关联：

	//src/components/Voting.jsx

	import React from 'react/addons';
	import {connect} from 'react-redux';
	import Winner from './Winner';
	import Vote from './Vote';
	import * as actionCreators from '../action_creators';

	export const Voting = React.createClass({
	  mixins: [React.addons.PureRenderMixin],
	  render: function() {
	    return <div>
	      {this.props.winner ?
	        <Winner ref="winner" winner={this.props.winner} /> :
	        <Vote {...this.props} />}
	    </div>;
	  }
	});

	function mapStateToProps(state) {
	  return {
	    pair: state.getIn(['vote', 'pair']),
	    hasVoted: state.get('hasVoted'),
	    winner: state.get('winner')
	  };
	}

	export const VotingContainer = connect(
	  mapStateToProps,
	  actionCreators
	)(Voting);

这么配置后，我们的`Voting`组件的`vote`参数属性将会与`vote`aciton creator关联起来。这样当点击
某个投票按钮后，会导致触发`VOTE`动作。

### 5.12 使用Redux Middleware发送actions到服务端

最后我们要做的是把用户数据提交到服务端，这种操作一般发生在用户投票，或选择跳转下一轮投票时发生。

让我们讨论一下投票操作，下面列出了投票的逻辑：

- 当用户进行投票，`VOTE`action将产生并分派到客户端的Redux Store中；
- `VOTE`actions将触发客户端reducer进行`hasVoted`状态设置；
- 服务端监控客户端通过socket.io投递的`action`，它将接收到的actions分派到服务端的Redux Store;
- `VOTE`action将触发服务端的reducer，其会创建vote数据并更新对应的票数。

这样来说，我们似乎已经都搞定了。唯一缺少的就是让客户端发送`VOTE`action给服务端。这相当于两端的
Redux Store相互分派action，这就是我们接下来要做的。

那么该怎么做呢？Redux并没有内建这种功能。所以我们需要设计一下何时何地来做这个工作：从客户端发送
action到服务端。

Redux提供了一个通用的方法来封装action：[Middleware](http://rackt.github.io/redux/docs/advanced/Middleware.html)。

Redux中间件是一个函数，每当action将要被指派，并在对应的reducer执行之前会被调用。它常用来做像日志收集，
异常处理，修整action，缓存结果，控制何时以何种方式来让store接收actions等工作。这正是我们可以利用的。

注意，一定要分清Redux中间件和Redux监听器的差别：中间件被用于action将要指派给store阶段，它可以修改action对
store将带来的影响。而监听器则是在action被指派后，它不能改变action的行为。

我们需要创建一个“远程action中间件”，该中间件可以让我们的action不仅仅能指派给本地的store，也可以通过
socket.io连接派送给远程的store。

让我们创建这个中间件，It is a function that takes a Redux store, and returns another function that takes a "next" callback. That function returns a third function that takes a Redux action. The innermost function is where the middleware implementation will actually go
（译者注：这句套绕口，请看官自行参悟）：

	//src/remote_action_middleware.js

	export default store => next => action => {

	}

上面这个写法看着可能有点渗人，下面调整一下让大家好理解：

	export default function(store) {
		return function(next) {
			return function(action) {

			}
		}
	}

这种嵌套接受单一参数函数的写法成为[currying](https://en.wikipedia.org/wiki/Currying)。
这种写法主要用来简化中间件的实现：如果我们使用一个一次性接受所有参数的函数（`function(store, next, action) { }`），
那么我们就不得不保证我们的中间件具体实现每次都要包含所有这些参数。

上面的`next`参数作用是在中间件中一旦完成了action的处理，就可以调用它来退出当前逻辑：

	//src/remote_action_middleware.js

	export default store => next => action => {
	  return next(action);
	}

如果中间件没有调用`next`，则该action将丢弃，不再传到reducer或store中。

让我们写一个简单的日志中间件：

	//src/remote_action_middleware.js

	export default store => next => action => {
	  console.log('in middleware', action);
	  return next(action);
	}

我们将上面这个中间件注册到我们的Redux Store中，我们将会抓取到所有action的日志。中间件可以通过Redux
提供的`applyMiddleware`函数绑定到我们的store中：

	//src/components/index.jsx

	import React from 'react';
	import Router, {Route, DefaultRoute} from 'react-router';
	import {createStore, applyMiddleware} from 'redux';
	import {Provider} from 'react-redux';
	import io from 'socket.io-client';
	import reducer from './reducer';
	import {setState} from './action_creators';
	import remoteActionMiddleware from './remote_action_middleware';
	import App from './components/App';
	import {VotingContainer} from './components/Voting';
	import {ResultsContainer} from './components/Results';

	const createStoreWithMiddleware = applyMiddleware(
	  remoteActionMiddleware
	)(createStore);
	const store = createStoreWithMiddleware(reducer);

	const socket = io(`${location.protocol}//${location.hostname}:8090`);
	socket.on('state', state =>
	  store.dispatch(setState(state))
	);

	const routes = <Route handler={App}>
	  <Route path="/results" handler={ResultsContainer} />
	  <DefaultRoute handler={VotingContainer} />
	</Route>;

	Router.run(routes, (Root) => {
	  React.render(
	    <Provider store={store}>
	      {() => <Root />}
	    </Provider>,
	    document.getElementById('app')
	  );
	});

如果你重启应用，你将会看到我们设置的中间件会抓到应用触发的action日志。

那我们应该怎么利用中间件机制来完成从客户端通过socket.io连接发送action给服务端呢？在此之前我们肯定需要先
有一个连接供中间件使用，不幸的是我们已经有了，就在`index.jsx`中，我们只需要中间件可以拿到它即可。
使用currying风格来实现这个中间件很简单：

	//src/remote_action_middleware.js

	export default socket => store => next => action => {
	  console.log('in middleware', action);
	  return next(action);
	}

这样我们就可以在`index.jsx`中传入需要的连接了：

	//src/index.jsx

	const socket = io(`${location.protocol}//${location.hostname}:8090`);
	socket.on('state', state =>
	  store.dispatch(setState(state))
	);

	const createStoreWithMiddleware = applyMiddleware(
	  remoteActionMiddleware(socket)
	)(createStore);
	const store = createStoreWithMiddleware(reducer);

注意跟之前的代码比，我们需要调整一下顺序，让socket连接先于store被创建。

一切就绪了，现在就可以使用我们的中间件发送`action`了：

	//src/remote_action_middleware.js

	export default socket => store => next => action => {
	  socket.emit('action', action);
	  return next(action);
	}

打完收工。现在如果你再点击投票按钮，你就会看到所有连接到服务端的客户端的票数都会被更新！

还有个很严重的问题我们要处理：现在每当我们收到服务端发来的`SET_STATE`action后，这个action都将会直接回传给
服务端，这样我们就造成了一个死循环，这是非常反人类的。

我们的中间件不应该不加处理的转发所有的action给服务端。个别action，例如`SET_STATE`，应该只在客户端做
处理。我们在action中添加一个标识位用于识别哪些应该转发给服务端：

	//src/remote_action_middleware.js

	export default socket => store => next => action => {
	  if (action.meta && action.meta.remote) {
	    socket.emit('action', action);
	  }
	  return next(action);
	}

我们同样应该修改相关的action creators：

	//src/action_creators.js

	export function setState(state) {
	  return {
	    type: 'SET_STATE',
	    state
	  };
	}

	export function vote(entry) {
	  return {
	    meta: {remote: true},
	    type: 'VOTE',
	    entry
	  };
	}

让我们重新审视一下我们都干了什么：

1. 用户点击投票按钮，`VOTE`action被分派；
2. 远程action中间件通过socket.io连接转发该action给服务端；
3. 客户端Redux Store处理这个action，记录本地`hasVoted`属性；
4. 当action到达服务端，服务端的Redux Store将处理该action，更新所有投票及其票数；
5. 设置在服务端Redux Store上的监听器将改变后的状态数据发送给所有在线的客户端；
6. 每个客户端将触发`SET_STATE`action的分派；
7. 每个客户端将根据这个action更新自己的状态，这样就保持了与服务端的同步。

为了完成我们的应用，我们需要实现下一步按钮的逻辑。和投票类似，我们需要将数据发送到服务端：

	//src/action_creator.js

	export function setState(state) {
	  return {
	    type: 'SET_STATE',
	    state
	  };
	}

	export function vote(entry) {
	  return {
	    meta: {remote: true},
	    type: 'VOTE',
	    entry
	  };
	}

	export function next() {
	  return {
	    meta: {remote: true},
	    type: 'NEXT'
	  };
	}

`ResultsContainer`组件将会自动关联action creators中的next作为props：

	//src/components/Results.jsx

	import React from 'react/addons';
	import {connect} from 'react-redux';
	import Winner from './Winner';
	import * as actionCreators from '../action_creators';

	export const Results = React.createClass({
	  mixins: [React.addons.PureRenderMixin],
	  getPair: function() {
	    return this.props.pair || [];
	  },
	  getVotes: function(entry) {
	    if (this.props.tally && this.props.tally.has(entry)) {
	      return this.props.tally.get(entry);
	    }
	    return 0;
	  },
	  render: function() {
	    return this.props.winner ?
	      <Winner ref="winner" winner={this.props.winner} /> :
	      <div className="results">
	        <div className="tally">
	          {this.getPair().map(entry =>
	            <div key={entry} className="entry">
	              <h1>{entry}</h1>
	              <div className="voteCount">
	                {this.getVotes(entry)}
	              </div>
	            </div>
	          )}
	        </div>
	        <div className="management">
	          <button ref="next"
	                   className="next"
	                   onClick={this.props.next()}>
	            Next
	          </button>
	        </div>
	      </div>;
	  }
	});

	function mapStateToProps(state) {
	  return {
	    pair: state.getIn(['vote', 'pair']),
	    tally: state.getIn(['vote', 'tally']),
	    winner: state.get('winner')
	  }
	}

	export const ResultsContainer = connect(
	  mapStateToProps,
	  actionCreators
	)(Results);

彻底完工了！我们实现了一个功能完备的应用。

## 结语

这是使用React+Redux的完整的测试驱动的项目开发的全过程，里面涉及到很多的知识点和概念。如果对部分技术点存有疑惑，理解的不完整，可以参照我的博客中React课程系列文章进行学习，每部分均有示例项目和文章讲义。如有问题，请随时交流（Github：https://github.com/GuoYongfeng）。
