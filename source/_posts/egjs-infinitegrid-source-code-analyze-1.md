---
title: egjs-infinitegrid源码分析(一)
date: 2019-04-30 11:55:43
tags: [Javascript,JS,Frontend,Layout,Source Code Analyze ]
category: Egjs-InfiniteGrid Code Analyze
thumbnail: https://github.com/naver/egjs-infinitegrid/raw/master/demo/assets/image/layout.gif
---
## Egjs-InfiniteGrid 简介
[Egjs-InfiniteGrid](https://github.com/naver/egjs-infinitegrid/),是韩国Naver公司所开发的一个前端库，可用于实现各种布局，如经典的瀑布流，方块布局等，与传统的 [Masonary](https://masonry.desandro.com/layout.html) 相比其拥有更快的速度，更少的内存占用等优点。
![Egjs-InfiniteGrid Preview](https://github.com/naver/egjs-infinitegrid/raw/master/demo/assets/image/layout.gif)
<!-- More -->
## 项目分析
### Demo
可于项目主页处查阅相关Demo
### 依赖
整个IG项目只依赖一个库: [egjs-component](https://www.npmjs.com/package/@egjs/component) 其主要功能是提供基于事件的组件。通过 `Conponent.on()`函数来注册事件,`Component.trigger()`来触发事件，并提供相关的功能，实现较为简单。同时整个项目已完全 Typescript 化，代码结构清晰，类型完整。
### 结构
IG库包括:
* Layout-用于布局元素
* DOMRenderer-用于将元素渲染至DOM
* ItemManager-用于管理元素
* LayoutManager-用于调用Layout来布局元素
* Infinite-用于处理数据加载请求和元素回收
* Watcher-用于监听如滚动、窗口缩放等事件
* InfiniteGrid-用于统筹调用以上子模块来实现完整的请求-加载-布局-回收的循环
### 约定
为了便于理解和叙述方便，我们始终以自上而下的布局作为默认布局，不考虑水平布局，这对于某些变量名如 `Size` 更容易理解。
我们将分别介绍每个模块，并最后完整的走一遍循环。

### InfiniteGrid.constructor
简单了解一下 InfiniteGrid 的构造函数。
``` js
// element: 我们希望创建 IG 的元素, 此处始终设为 '.container'
// isOverflowScroll: 滚动容器的设置,在 Watcher.ts 详细介绍
// threshold: 加载阈值
// isEqualSize & isConstantSize: 不做讨论
// useRecycle: 使用回收, 不讨论为false的情况
// horizontal: 水平布局, 不做讨论
// transitionDuration: 动画时间
// useFit: 不做讨论
constructor(element: HTMLElement | string | IJQuery, options?: Partial<IInfiniteGridOptions>) {
	super();
	// 存储参数
	assign(this.options = {
		itemSelector: "*",
		isOverflowScroll: false,
		threshold: 100,
		isEqualSize: false,
		isConstantSize: false,
		useRecycle: true,
		horizontal: false,
		transitionDuration: 0,
		useFit: true,
		attributePrefix: "data-",
	}, options);
	DEFENSE_BROWSER && (this.options.useFit = false);
	IS_ANDROID2 && (this.options.isOverflowScroll = false);
	this._reset();
	this._loadingBar = {};
	const {
		isOverflowScroll,
		isEqualSize,
		isConstantSize,
		horizontal,
		threshold,
		useRecycle,
	} = this.options;
	// 创建 元素管理模块
	this._items = new ItemManager();
		this._renderer = new DOMRenderer(element, {
			isEqualSize,
			isConstantSize,
			horizontal,
			container: isOverflowScroll,
		});
	// 创建 事件监听模块
	this._watcher = new Watcher(
		this._renderer.view,
		{
			isOverflowScroll,
			horizontal,
			container: this._renderer.container,
			resize: () => this._onResize(),
			check: param => this._onCheck(param),
		});
	// 创建 循环管理模块
	this._infinite = new Infinite(this._items, {
		useRecycle,
		threshold,
		append: param => this._requestAppend(param),
		prepend: param => this._requestPrepend(param),
		recycle: param => this._recycle(param),
	});
}
```