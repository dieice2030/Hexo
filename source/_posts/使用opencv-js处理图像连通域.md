---
title: 使用opencv.js处理图像连通域
date: 2024-12-03 20:05:59
tags:
- 前端
- 图像处理
- OpenCV
- OpenCV.js
---

项目上用到了segmentAnything，得到的蒙版有可能会有一些小的干扰区域，这个时候可以通过计算连通域，剔除掉脱离主体的干扰部分，主要用到opencv的 [connectedComponentsWithStats](https://docs.opencv.org/3.4/d3/dc0/group__imgproc__shape.html#gae57b028a2b2ca327227c2399a9d53241)函数

js中的函数声明
```ts
/**
 * This is an overloaded member function, provided for convenience. It differs from the above function
 * only in what argument(s) it accepts.
 *
 * @param image the 8-bit single-channel image to be labeled
 *
 * @param labels destination labeled image
 *
 * @param stats statistics output for each label, including the background label, see below for
 * available statistics. Statistics are accessed via stats(label, COLUMN) where COLUMN is one of
 * ConnectedComponentsTypes. The data type is CV_32S.
 *
 * @param centroids centroid output for each label, including the background label. Centroids are
 * accessed via centroids(label, 0) for x and centroids(label, 1) for y. The data type CV_64F.
 *
 * @param connectivity 8 or 4 for 8-way or 4-way connectivity respectively
 *
 * @param ltype output image label type. Currently CV_32S and CV_16U are supported.
 */
export declare function connectedComponentsWithStats(image: InputArray, labels: OutputArray, stats: OutputArray, centroids: OutputArray, connectivity?: int, ltype?: int): int;
```

假设输入是1024*768的图像  
`image` 是输入图像, Uint8的Mat，尺寸为1024\*768\*4  
`labels` 是输出结果，Int32的Mat， 尺寸为1024\*768  
`stats` 是每个连通域的信息，大小为 `连通域数量`\* 5，每个`row`分别是长度为5的数组，分别记录了连通域外接矩形左上角像素点的`x`，`y`，`宽度`，`高度`和`连通域的面积`  
`centroids` 是连通域的中心点坐标

```TS
  // 1. 将原始蒙版转化为灰度图像
  const grayIm = new cv.Mat();
  cv.cvtColor(im, grayIm, cv.COLOR_BGR2GRAY);
  // 2. 得到连通域
  const labels = new cv.Mat();
  const stats = new cv.Mat();
  const centroids = new cv.Mat();
  const cnt = cv.connectedComponentsWithStats(grayIm, labels, stats, centroids, 8);
  const miniAreaIndexes = [];
  // 3. 计算总面积
  let totalArea = 0;
  for (let i = 0; i < cnt; i++) {
    const area = stats.row(i).data32S[4];
    if (i !== 0) totalArea += area;
  }
  // 4. 记录需要过滤的连通域index
  for (let i = 1; i < cnt; i++) {
    const area = stats.row(i).data32S[4];
    // stats.row(0)表示的是整个原始蒙版背景
    // 这里判断需要过滤的连通域标准是：面积小于原始蒙版面积的1%且小于所有连通域平均面积的10%
    if (area < stats.row(0).data32S[4] * 0.01 && area < (totalArea / cnt) * 0.1) {
      miniAreaIndexes.push(i);
    }
  }
  //5.将原始蒙版中不符合要求的部分剔除
  for (let i = 0; i < labels.data32S.length; i++) {
    const label = labels.data32S.at(i);
    if (miniAreaIndexes.includes(label) && label !== 0) {
      im.data.set([0, 0, 0, 0], i * 4);
    }
  }
```



