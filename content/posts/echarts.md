---
title: ECharts使用心得
description: ECharts 折线图配置与使用心得。
publishDate: 2015-04-01 12:32:18
tags:
  - ECharts
  - jekyll
---

### [ECharts折线图实例](http://echarts.baidu.com/doc/example.html)

### 注意事项：

- 所有charts中，`legend`中`data`需要与`series` 中`name`完全相同,否则不能正确显示
- 如果**标准折线图**`series`中包含`stack`属性,则标准折线图会成为**堆积折线图**,即数据从上一个数据Y轴开始累积,不从Y轴0坐标开始,反之亦然

---

#### 折线图位置定义

![折线图位置定义](http://echarts.baidu.com/doc/asset/img/doc/grid.jpg)

- 其中`x`确定左边距，`y`确定上边距，`x2`确定右边距，`y2`确定下边距
- 当`y2`被定义，且图例`legend`位置为`bottom`时，可能会导致图例与坐标重叠
- 若x轴最右部分不能完全显示时，可适当增大`x2`值，直至完全显示
