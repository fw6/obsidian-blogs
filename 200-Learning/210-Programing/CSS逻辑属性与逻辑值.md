---
title: "CSS逻辑属性与逻辑值"
description: ""
pubDate: "2023-08-13 15:35"
heroImage: "https://images.unsplash.com/photo-1637729960394-1a1200764aa4?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1200"
date created: "2023-08-13 15:35"
date modified: "2023-08-13"
tags:
    - writings
    - CSS
    - W3C
---

- [CSS logical properties and values | MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_logical_properties_and_values/Basic_concepts_of_logical_properties_and_values)
- [Logical Properties - Learn CSS | web.dev](https://web.dev/learn/css/logical-properties/)

![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230813153919.png)

逻辑属性与逻辑值规范为 CSS 中的许多属性和值引入了相对于流的对应关系。

## 为何需要逻辑属性？

传统CSS根据实体尺度属性设置尺寸，例如一个块级盒子具有width、height，元素定位、内外边距、边框等按照方向划分为top、right、bottom、left等等。

实体属性在更改了书写模式（writing-mode）情况下，就无法适用了。如：
![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230813155017.png)
把writing-mode改为vertical-rl就会变为：
![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230813155049.png)
宽高比显然是不符合的。

如果使用逻辑属性（将width改为inline-size），就会变为：
![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230813155125.png)
这样无论何种书写模式都是正常的。

逻辑属性和逻辑规范定义了这些实体值到逻辑值的对应关系，如：
- left/right或top/bottom，对应逻辑值start和end

## 块向与行向尺度

在对齐元素时，诸如弹性盒和网格布局等新的 CSS 布局方法所用的概念是块向（`block`）和行向（`inline`），而非右侧（`right`）和左侧（`left`），或者上侧（`top`）和下侧（`bottom`）。

行向是指在所用的书写模式中，一行文本延伸的方向。因此在从左到右横排的英文文档，或者从右到左横排的阿拉伯文文档中，行向为_水平方向_。若切换至竖排书写模式（如日文文档）则行向变为_竖直方向_，这是因为文本在这种书写模式中竖直延伸。

块向是指另一方向，即块——例如段落——依次显示的方向。在英文和阿拉伯文中，块沿竖直方向依次排列，而块在任意竖排书写模式中沿水平方向依次排列。

下图展示了横排书写模式中的行向和块向：
![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230813155413.png)

此图展示了竖排书写模式中的块向与行向：
![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230813155513.png)

## 逻辑属性与实体属性对应关系

### 尺寸

|逻辑属性|实体属性|
|---|---|
|[`inline-size`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/inline-size)|[`width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/width)|
|[`block-size`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/block-size)|[`height`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/height)|
|[`min-inline-size`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/min-inline-size)|[`min-width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/min-width)|
|[`min-block-size`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/min-block-size)|[`min-height`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/min-height)|
|[`max-inline-size`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/max-inline-size)|[`max-width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/max-width)|
|`max-block-size`|[`max-height`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/max-height)|


### 外边距、边框和内边距

|逻辑属性|实体属性|
|---|---|
|[`border-block-end`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-block-end)|[`border-bottom`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-bottom)|
|[`border-block-end-color`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-block-end-color)|[`border-bottom-color`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-bottom-color)|
|[`border-block-end-style`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-block-end-style)|[`border-bottom-style`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-bottom-style)|
|[`border-block-end-width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-block-end-width)|[`border-bottom-width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-bottom-width)|
|[`border-block-start`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-block-start)|[`border-top`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-top)|
|[`border-block-start-color`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-block-start-color)|[`border-top-color`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-top-color)|
|[`border-block-start-style`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-block-start-style)|[`border-top-style`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-top-style)|
|[`border-block-start-width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-block-start-width)|[`border-top-width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-top-width)|
|[`border-inline-end`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-inline-end)|[`border-right`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-right)|
|[`border-inline-end-color`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-inline-end-color)|[`border-right-color`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-right-color)|
|[`border-inline-end-style`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-inline-end-style)|[`border-right-style`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-right-style)|
|[`border-inline-end-width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-inline-end-width)|[`border-right-width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-right-width)|
|[`border-inline-start`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-inline-start)|[`border-left`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-left)|
|[`border-inline-start-color`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-inline-start-color)|[`border-left-color`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-left-color)|
|[`border-inline-start-style`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-inline-start-style)|[`border-left-style`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-left-style)|
|[`border-inline-start-width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-inline-start-width)|[`border-left-width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-left-width)|
|[`border-start-start-radius`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-start-start-radius)|[`border-top-left-radius`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-top-left-radius)|
|[`border-end-start-radius`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-end-start-radius)|[`border-bottom-left-radius`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-bottom-left-radius)|
|[`border-start-end-radius`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-start-end-radius)|[`border-top-right-radius`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-top-right-radius)|
|[`border-end-end-radius`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-end-end-radius)|[`border-bottom-right-radius`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-bottom-right-radius)|
|[`margin-block-end`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/margin-block-end)|[`margin-bottom`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/margin-bottom)|
|[`margin-block-start`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/margin-block-start)|[`margin-top`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/margin-top)|
|[`margin-inline-end`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/margin-inline-end)|[`margin-right`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/margin-right)|
|[`margin-inline-start`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/margin-inline-start)|[`margin-left`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/margin-left)|
|[`padding-block-end`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/padding-block-end)|[`padding-bottom`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/padding-bottom)|
|[`padding-block-start`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/padding-block-start)|[`padding-top`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/padding-top)|
|[`padding-inline-end`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/padding-inline-end)|[`padding-right`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/padding-right)|
|[`padding-inline-start`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/padding-inline-start)|[`padding-left`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/padding-left)|

还有一些简写属性可以用于让我们同时设置盒子块向或者行向的两侧。这些简写属性没有等价的实体属性。

| 属性                                                                                          | 用途                                                                                                                                                                                                                                                                     |
| --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [`border-block`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-block)               | 为两个块向边框设置 [`border-color`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-color)、[`border-style`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-style) 和 [`border-width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-width)。 |
| [`border-block-color`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-block-color)   | 为两个块向边框设置 `border-color`。                                                                                                                                                                                                                                      |
| [`border-block-style`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-block-style)   | 为两个块向边框设置 `border-style`。                                                                                                                                                                                                                                      |
| [`border-block-width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-block-width)   | 为两个块向边框设置 `border-width`。                                                                                                                                                                                                                                      |
| [`border-inline`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-inline)             | 为两个行向边框设置 `border-color`、`border-style` 和 `border-width`。                                                                                                                                                                                                    |
| [`border-inline-color`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-inline-color) | 为两个行向边框设置 `border-color`。                                                                                                                                                                                                                                      |
| [`border-inline-style`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-inline-style) | 为两个行向边框设置 `border-style`。                                                                                                                                                                                                                                      |
| [`border-inline-width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-inline-width) | 为两个行向边框设置 `border-width`。                                                                                                                                                                                                                                      |
| [`margin-block`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/margin-block)               | 设置所有块向外边距（[`margin`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/margin)）。                                                                                                                                                                              |
| [`margin-inline`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/margin-inline)             | 设置所有行向外边距（`margin`）。                                                                                                                                                                                                                                         |
| [`padding-block`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/padding-block)             | 设置块向内边距（[`padding`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/padding)）。                                                                                                                                                                                |
| [`padding-inline`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/padding-inline)           | 设置行向内边距（`padding`）。                                                                                                                                                                                                                                            |
|                                                                                               |                                                                                                                                                                                                                                                                          |

### 浮动和定位

|逻辑属性或逻辑值|实体属性或实体值|
|---|---|
|[`float`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/float): inline-start|[`float`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/float): left|
|[`float`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/float): inline-end|[`float`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/float): right|
|[`clear`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/clear): inline-start|[`clear`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/clear): left|
|[`clear`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/clear): inline-end|[`clear`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/clear): right|
|[`inset-inline-start`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/inset-inline-start)|[`left`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/left)|
|[`inset-inline-end`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/inset-inline-end)|[`right`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/right)|
|[`inset-block-start`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/inset-block-start)|[`top`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/top)|
|[`inset-block-end`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/inset-block-end)|[`bottom`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/bottom)|
|[`text-align`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/text-align): start|[`text-align`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/text-align): left|
|[`text-align`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/text-align): end|[`text-align`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/text-align): right|

除了这些有对应关系的属性，另有可以处理块向和行向尺度的简写属性。除 [`inset`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/inset) 属性外，这些属性没有与实体属性的对应关系。

|逻辑属性|用途|
|---|---|
|[`inset-inline`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/inset-inline)|为行向尺度同时设置上述两个偏移值。|
|[`inset-block`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/inset-block)|为块向尺度同时设置上述两个偏移值。|
|[`inset`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/inset)|同时设置四个偏移值，多值语法使用实体对应关系。|
