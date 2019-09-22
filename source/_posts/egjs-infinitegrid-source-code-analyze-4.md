---
title: egjs-infinitegrid源码分析(四):GridLayout
date: 2019-05-03 17:11:50
tags: [Javascript,JS,Frontend,Layout,Source Code Analyze ]
category: Egjs-InfiniteGrid Code Analyze
---
## GridLayout
这节我们分析一下 `GridLayout` 的源码, 和 `GridLayout` 类似的还有 `JustifiedLayout` 等.
虽然是 Layout 但其实仍然没有直接修改视图,而是根据 `outline`, 来计算每个元素的位置. 
并且每个元素总是插入最短的一列,来维持整体的平衡.
<!-- More -->
## GridLayout 类属性
```ts
public options: {
    horizontal: boolean,
    margin: number, // margin 值
    align: IAlign[keyof IAlign], // 对齐方式,我们只考虑 center 居中
    itemSize: number, // 元素的大小(宽度)
};

// _size 当前可视区域的宽度
private _size: number; 

// _columnLength 每列的宽度
private _columnSize: number;

// _columnLength 列数
private _columnLength: number;

// _style 根据 horizontal.horizontal 计算出的一些属性名
// 我们只考虑为 false 时，其属性包括
// startPos1: "top",
// endPos1: "bottom",
// size1: "height",
// startPos2: "left",
// endPos2: "right",
// size2: "width",
private _style: IRectlProperties;
```
## 函数 constructor
``` ts
constructor(options: Partial<GridLayout["options"]> = {}) {
    this.options = assignOptions({
        margin: 0,
        horizontal: false,
        align: START,
        itemSize: 0,
    }, options);
    this._size = 0;
    this._columnSize = 0;
    this._columnLength = 0;
    this._style = getStyleNames(this.options.horizontal);
}
```
## 函数 append & prepend
``` ts
// 这两个函数都调用了 _insert 函数，并通过 APPEND|PREPEND (true/false) 来区分
public append(items: IInfiniteGridItem[], outline?: number[], cache?: boolean) {
    return this._insert(items, outline, APPEND, cache);
}
public prepend(items: IInfiniteGridItem[], outline?: number[], cache?: boolean) {
    return this._insert(items, outline, PREPEND, cache);
}
```
## 函数 _insert
``` ts
private _insert(
    items: IInfiniteGridItem[] = [],
    outline: number[] = [],
    isAppend?: boolean,
    cache?: boolean,
) {
    const clone = cache ? items : cloneItems(items);
    // 上一组的 outline
    let startOutline = outline;
    // 更新 列数
    if (!this._columnLength) {
        this.checkColumn(items[0]);
    }
    // 如果列数不匹配则重新填充上一组 outline，
    // 比如在第一次布局时因为没有上一组所以 outline 为 [0], 此时需要根据列数修改 outlien
    if (outline.length !== this._columnLength) {
        startOutline = fill(new Array(this._columnLength), outline.length ? (Math[isAppend ? "min" : "max"](...outline) || 0) : 0);
    }
    // 调用 _layout 函数
    const result = this._layout(clone, startOutline, isAppend);

    return {
        items: clone,
        outlines: result,
    };
}
```
## 函数 _layout
``` ts
private _layout(items: IInfiniteGridItem[], outline: number[], isAppend?: boolean) {
    const length = items.length;
    const margin = this.options.margin;
    const align = this.options.align;
    const style = this._style;

    const size1Name = style.size1; // "height"
    const size2Name = style.size2; // "width"
    const pos1Name = style.startPos1; // "top"
    const pos2Name = style.startPos2; // "bottom"
    const columnSize = this._columnSize;
    const columnLength = this._columnLength;
    // 上面一大段都是为了方便调用

    const size = this._size;
    // viewDist 完全排列后可视区域水平剩余空间
    const viewDist = (size - (columnSize + margin) * columnLength + margin);

    const pointCaculateName = isAppend ? "min" : "max";
    const startOutline = outline.slice();
    const endOutline = outline.slice();

    for (let i = 0; i < length; ++i) {
        // 找到最短的一列
        const point = Math[pointCaculateName](...endOutline) || 0;
        let index = endOutline.indexOf(point);

        const item = items[isAppend ? i : length - 1 - i]; // 获取第一个元素
        const size1 = item.size[size1Name]; // height
        const size2 = item.size[size2Name]; // width
        const pos1 = isAppend ? point : point - margin - size1; // 根据当前列的底部值，计算新元素顶部位置值
        const endPos1 = pos1 + size1 + margin; // 计算底部位置值

        if (index === -1) {
            index = 0;
        }
        let pos2 = (columnSize + margin) * index; // 计算左侧位置

        // ALIGN
        if (align === CENTER) {
            pos2 += viewDist / 2; // 对齐，向右移动 viewDist / 2 保持居中
        } /* else if 其他对齐 */
        // 更新元素位置
        item.rect = {
            [pos1Name as "top"]: pos1,
            [pos2Name as "left"]: pos2,
        };
        item.column = index;
        endOutline[index] = isAppend ? endPos1 : pos1; // 更新复制的outline，作为新的 outline
    }
    if (!isAppend) {
        /* unreleated code */
    }
    // if append items, startOutline is low, endOutline is high
    // if prepend items, startOutline is high, endOutline is low
    return {
        start: isAppend ? startOutline : endOutline,
        end: isAppend ? endOutline : startOutline,
    };
}
```
## 函数 layout
layout 函数用于对多组 Group 进行布局
``` ts
public layout(groups: IInfiniteGridGroup[] = [], outline: number[] = []) {
    const firstItem = (groups.length && groups[0].items.length && groups[0].items[0]);

    this.checkColumn(firstItem);

    // if outlines' length and columns' length are not same, re-caculate outlines.
    let startOutline: number[];

    if (outline.length !== this._columnLength) {
        const pos = outline.length === 0 ? 0 : Math.min(...outline);

        // re-layout items.
        startOutline = fill(new Array(this._columnLength), pos);
    } else {
        startOutline = outline.slice();
    }
    groups.forEach(group => {
        const items = group.items;
        // 调用 _layout 进行布局
        const result = this._layout(items, startOutline, APPEND);

        group.outlines = result;
        startOutline = result.end;
    });

    return this;
}
```