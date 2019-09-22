---
title: egjs-infinitegrid源码分析(二):Watcher 
date: 2019-05-01 15:43:19
tags: [Javascript,JS,Frontend,Layout,Source Code Analyze ]
category: Egjs-InfiniteGrid Code Analyze
thumbnail: https://s2.ax1x.com/2019/05/01/EJqM60.png
---
## Watcher
> 先从软柿子开始捏，代码文件为 `./src/Watcher.ts`

Watcher 模块主要用于提供事件的监听，如浏览器窗口缩放，滚动等事件
<!-- More -->
## 滚动区域
IG 支持两种滚动模式，并通过 isOverflowScroll 来设定:
1. `false`: 使用窗口window 作为滚动组件
2. `true`: 使用 某个div(class:container) 作为滚动组件
![区别](https://s2.ax1x.com/2019/05/01/EYeohd.png)

## 引用
``` js
import {
	IS_IOS,
} from "./consts";
import {
	window,
} from "./browser";
import {
	addEvent,
	removeEvent,
	scroll,
	scrollTo,
	scrollBy,
	assign,
} from "./utils";
import { WindowMockType, IWatchStatus } from "./types";
```
引用部分主要引用了一些工具函数和一些常量，之所以将这些工具函数独立出来是考虑各版本浏览器兼容性的问题，考虑到文章主题，我们并不打算详细讨论这些问题，仅仅使用一个示例来讲解
### 兼容函数 addEvent
``` ts
export function addEvent(
    //支持的对象有普通元素或窗口对象
	element: Element | WindowMockType, 
	type: string,
	handler: (...args: any[]) => any,
	eventListenerOptions?: boolean | { [key: string]: any },
) {
	if (SUPPORT_ADDEVENTLISTENER) {
        // 支持 addEventListener 函数的浏览器
		let options = eventListenerOptions || false;

		if (typeof eventListenerOptions === "object") {
			options = SUPPORT_PASSIVE ? eventListenerOptions : false;
		}
		element.addEventListener(type, handler, options);
	} else if ((element as any).attachEvent) {
        // 支持 attachEvent 函数的浏览器
		(element as any).attachEvent(`on${type}`, handler);
	} else {
        // 需要手动添加的浏览器
		(element as any)[`on${type}`] = handler;
	}
}
```
## 类属性
``` ts
// 选项
public options: IWatcherOptions;
export interface IWatcherOptions {
    // 包含全部元素的组件
    // isisOverflowScroll 为 false 时为 class:container
    // isisOverflowScroll 为 true 时为 class:_eg-infinitegrid-container_
	container: HTMLElement; 
	isOverflowScroll: boolean; // 滚动组件设定选项
	horizontal: boolean; // 是否水平布局
	resize: () => void; // 窗口缩放handler
	check: (e?: {
		isForward: boolean,
		scrollPos: number,
		orgScrollPos: number,
		horizontal: boolean,
	}) => void; // check函数会在鼠标滚动时调用
}
// 用于处理窗口缩放事件多次触发->只在"最后一次"触发后调用相关函数
private _timer: {
    resize: any;
};
// _containerOffset 是指 container 到 _view 顶部的距离，始终固定
// ----------- _view
//            | _containerOffset
// ----------- container
private _containerOffset: number;
// 用于监听滚动事件的元素
// isisOverflowScroll 为 false 时为 window
// isisOverflowScroll 为 true 时为 class:container
private _view: WindowMockType | HTMLElement;
// 不做讨论
private _isScrollIssue: boolean;
// 为 container 基于 _view 的滚动位置
private _prevPos: number;
```
## 构造函数
``` ts
	constructor(view: WindowMockType | HTMLElement, options: Partial<IWatcherOptions> = {}) {
        // 导入选项
		assign(this.options = {
			container: view as HTMLElement,
			resize: () => void 0,
			check: () => void 0,
			isOverflowScroll: false,
			horizontal: false,
		}, options);
		this._timer = {
			resize: null,
		};
        // 初始化，这里仅初始化了 this._prevPos=null;
		this.reset();
		this._containerOffset = 0;
		this._view = view;
		this._isScrollIssue = IS_IOS;
        // 将函数绑定到自身
		this._onCheck = this._onCheck.bind(this);
		this._onResize = this._onResize.bind(this);
        // 添加 resize 和 scroll 事件监听
		this.attachEvent();
		this.resize();
		this.setScrollPos();
	}
```
## 函数 resize
``` ts
public resize() {
    // 修正 this._containerOffset
    // isOverflowScroll 为 true 时，container 是自动生成并填充 _view 的，因此 offset = 0
	this._containerOffset = this.options.isOverflowScroll ? 0 : this._getOffset();
}
private _getOffset() {
	const { container, horizontal } = this.options;
	const rect = container.getBoundingClientRect();
    // rect["top"] 即为 container 到窗口可视区域顶部的距离，当container处于窗口顶部以下时该值为正，否则该值为负值。最后相加结果始终固定。
	return rect[horizontal ? "left" : "top"] + this.getOrgScrollPos();
}
// 获取滚动位置
public getOrgScrollPos() {
    // isOverflowScroll 为 false 时即为窗口当前滚动条的位置
    // isOverflowScroll 为 true 时即为 class:container 的滚动条位置
    return scroll(this._view, this.options.horizontal);
}
```
## 函数 _onResize _onCheck
``` ts
public attachEvent() {
	addEvent(this._view, "scroll", this._onCheck);
	addEvent(window, "resize", this._onResize);
}
```

``` ts
private _onResize() {
    // 多次触发处理
	if (this._timer.resize) {
		clearTimeout(this._timer.resize);
	}
	this._timer.resize = setTimeout(() => {
		this.resize();
		this.options.resize(); // 调用实际处理函数
		this._timer.resize = null;
		this.reset();
	}, 100);
}
public reset() {
	this._prevPos = null;
}
```

``` ts
private _onCheck() {
	const prevPos = this.getScrollPos();
	const orgScrollPos = this.getOrgScrollPos();
    // 更新滚动位置
	this.setScrollPos(orgScrollPos);
	const scrollPos = this.getScrollPos();
	if (prevPos === null || (this._isScrollIssue && orgScrollPos === 0) || prevPos === scrollPos) {
		orgScrollPos && (this._isScrollIssue = false);
		return;
	}
	this._isScrollIssue = false;
    // 调用实际处理函数
	this.options.check({
		isForward: prevPos < scrollPos,
		scrollPos,
		orgScrollPos,
		horizontal: this.options.horizontal,
	});
}

public setScrollPos(pos = this.getOrgScrollPos()) {
	let rawPos = pos;
	if (typeof pos === "undefined") {
		rawPos = this.getOrgScrollPos();
	}
    // 为container 基于 _view 的滚动位置
	this._prevPos = rawPos - this.getContainerOffset();
}
```
## 函数 getStatus setStatus scrollBy scrollTo
这些函数功能都较为简单，不做详细介绍
``` ts
public getStatus(): IWatchStatus {
	return {
		_prevPos: this._prevPos,
		scrollPos: this.getOrgScrollPos(),
	};
}
public setStatus(status: IWatchStatus, applyScrollPos = true) {
	this._prevPos = status._prevPos;
	applyScrollPos && this.scrollTo(status.scrollPos);
}
public scrollBy(pos: number) {
    const arrPos = this.options.horizontal ? [pos, 0] : [0, pos];
    
	scrollBy(this._view, arrPos[0], arrPos[1]);
	this.setScrollPos();
}
public scrollTo(pos: number) {
    const arrPos = this.options.horizontal ? [pos, 0] : [0, pos];
    
	scrollTo(this._view, arrPos[0], arrPos[1]);
}
```

