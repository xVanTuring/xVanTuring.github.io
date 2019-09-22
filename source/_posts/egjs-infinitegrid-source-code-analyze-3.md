---
title: egjs-infinitegrid源码分析(三):ItemManager
date: 2019-05-03 15:14:09
tags: [Javascript,JS,Frontend,Layout,Source Code Analyze ]
category: Egjs-InfiniteGrid Code Analyze
---
## ItemManager
`ItemManager` 用于管理添加删除元素，并提供各类数据
## 类型 IInfiniteGridItem
``` ts
// 文件位置: ./src/type.ts
export interface IInfiniteGridItem {
	groupKey: string | number;
	content: string;
	el?: IInfiniteGridItemElement;
	orgSize?: ISize;
	size?: ISize;
	rect?: Partial<ISize & IPosition>;
	prevRect?: Partial<ISize & IPosition>;
	[key: string]: any;
}
```
<!-- More -->
```ts
export interface IInfiniteGridItemElement extends HTMLElement {
	_INFINITEGRID_TRANSITION?: boolean;
	__IMAGE__?: -1 | IInfiniteGridItemElement;
	__BOX__?: IInfiniteGridItemElement;
	__SIZE__?: number;
	__TRANSLATE__?: number;
	__RATIO__?: number;
}
```
每个存储于IG的元素都使用该类型,其中
* groupKey: 表示所在组的键
* content: 用于存储 html 字符串 
* el: 实际上是 HTMLElement,但多了几项数据
* orgSize: 元素第一次渲染出来的大小
* size: 元素现在的大小
* rect: 元素的位置,包含 top 和 left
* prevRect: 上一次的位置

## 类型 IInfiniteGridGroup
> 上面提到每个元素都有一个 groupKey 用于表示所在的组,该组即为 `IInfiniteGridGroup`
``` ts
export interface IInfiniteGridGroup {
	groupKey: string | number;
	items: IInfiniteGridItem[];
	outlines: { start: number[], end: number[] };
}
```
每个组包含了一下信息:
* groupKey: 该组的键
* items: 存储所包含的 Item 数组
* outlines.start: 用于存储该组起始位置值,以4列瀑布流为例,其为包含四个数字的数组,值为该组在各列的第一个元素的 top 值. 
* outlines.end: 同上. 

## 静态函数 ItemManager.from
``` ts
public static from(
	elements: HTMLElement[] | string | string[] | IJQuery, selector: string,
	{ groupKey }: { groupKey: string | number }) {
	const filted = ItemManager.selectItems($(elements, MULTI), selector);
	// Item Structure
	return toArray(filted).map(el => ({
		el,
		groupKey,
		content: el.outerHTML,
		rect: {
			top: DUMMY_POSITION, // -100000
			left: DUMMY_POSITION,
		},
	}));
}
```
该函数用于将 HTML 模板转化为我们所需要的 IInfiniteGridItem 数据,并为其填充默认值.
代码中的 `$` 函数并非为 jquery 而是出于习惯命名的, 其功能与 jquery 类似, 如果提供的是 HTML 代码则会自动创建,  如果是 selector 则会自动选择.
`ItemManager.selectItems` 用于过滤元素, 通过提供的 selector (默认为'`*`'),对元素类名进行过滤
## ItemManager 属性 & constructor
该类仅有一个 `_data` 属性类型为 `IInfiniteGridGroup`, 并由 constructor 函数初始化
``` ts
public _data: IInfiniteGridGroup[];
constructor() {
	this.clear(); // this._data = [];
}
```
## 函数 append & prepend
> 代码较为简单不做解释
```ts
public append(layouted: IInfiniteGridGroup) {
	this._data.push(layouted);
	return layouted.items;
}
public prepend(layouted: IInfiniteGridGroup) {
	this._data.unshift(layouted);
	return layouted.items;
}
```
## 函数 get & set
```ts
public get(start?: number, end?: number) {
    // 判断是否返回整个数组,或子数组.
    // 如 end 为 underfine 则只会返回 `start` 位置的一个元素数组
	return isUndefined(start) ? this._data :
		this._data.slice(start, (isUndefined(end) ? start : end) + 1);
}
```

```ts
// 设置整个 _data ,或者设置 key 所指元素
public set(data: IInfiniteGridGroup | IInfiniteGridGroup[], key?: string | number);
```
## 函数 plunk
该函数用于从 _data 提取每个 propery 数据 存储为数组
```ts
public pluck<T extends keyof IInfiniteGridGroup>(property: T, start?: number, end?: number)
```
## 函数 getEdgeIndex && getEdgeValue
搞笑的是这两个函数根本没用到过, 和它们同名的函数在其他模块被重新实现并调用了.....
## 函数 getMaxEdgeValue
用于获取终端处的最大值,从后往前搜索最大 `outlines.end` 值
``` ts
public getMaxEdgeValue() {
    const groups = this.get();
    const length = groups.length;

    for (let i = length - 1; i >= 0; --i) {
        const end = groups[i].outlines.end;

        if (end.length) {
            const pos = Math.max(...end);

            return pos;
        }
    }
    return 0;
}
```
## 其他
除了这些函数以外 `ItemManager` 还包含了其他的一些简单函数如 `remove`,`indexOf`. 
整个 `ItemManager` 还是很简单的.