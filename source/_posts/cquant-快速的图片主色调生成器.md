---
title: 'CQuant: 快速的图片主色调生成器'
date: 2019-04-30 21:28:14
tags: [Native,  JS,  Node.js Addon, N-API]
thumbnail: https://s2.ax1x.com/2019/04/30/EGRnCF.png
---
## 起因
> 在做Sparrow项目的时候需要实现通过颜色来搜索图片, 最好的方法当然是获取图片的主色调然后通过颜色比较来过滤. 

NPM倒是有[纯JS的实现](https://www.npmjs.com/package/image-palette), 算法是MMCQ, 不过效率实在是不咋地, 一张1920x1080的图片需要消耗 950ms 将近 1S(????)的时间, 最重要的是这个代码会阻塞主线程就很难受了.<!-- more -->所以在查阅相关算法资料后便开始自己造了.
## 预览
![Screenshot from 2019-02-09 15-16-32.png](https://s2.ax1x.com/2019/04/30/EGRnCF.png)
## 上手
### · 安装
``` bash
npm i cquant sharp // install cquant and sharp
```
### · 使用
``` js
const cquant = require('cquant')
const sharp = require('sharp')
sharp('path/to/image.png')
  .raw() // convert raw buffer like [RGB RGB RGB RGB]
  .toBuffer((_err,  buffer,  info) => {
    if (!_err) {
      let colorCount = 4
      cquant.paletteAsync(buffer,  info.channels,  colorCount).then(res => {
        //结果
        console.log(res)
      }).catch(err => {
        console.log(err)
      })
    }
  })
```
### · 优点
1. 很快
2. 省内存
3. 异步, 可以同时处理多个图片也不会阻塞主线程

### · 性能
#### JPG 1920 x 1280 (No SubSample)
| Program       | Time(ms) |
|---------------|:--------:|
| cquant        |   60 ms  |
| image-palette |    N/A   |
#### JPG 1920 x 1280 (No SubSample)
| Program       | Time(ms) |
|---------------|:--------:|
| cquant        |   12ms   |
| image-palette |   950ms  |
> ps: image-palette 完全无法处理大图片, Node会直接Crash掉
## Extra
* Repo: [cquant](https://github.com/xVanTuring/cquant)
* Site: [leptonica](http://www.leptonica.com/)