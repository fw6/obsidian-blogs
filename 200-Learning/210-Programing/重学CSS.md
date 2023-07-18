---
title: "重学CSS"
description: ""
pubDate: "2023-06-27 16:23"
heroImage: "https://images.unsplash.com/photo-1523437113738-bbd3cc89fb19?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=640&q=80"
date created: "2023-06-26 09:42"
date modified: "2023-06-27"
tags:
    - notes
    - CSS
    - Programming
    - FE
---

#CSS #Programming #FE

> Silence is deep as Eternity; Speech is shallow as Time.
> — <cite>Thomas Carlyle</cite>



[Learn CSS | web.dev](https://web.dev/learn/css/)
[CSS - 学习 Web 开发 | MDN](https://developer.mozilla.org/zh-CN/docs/Learn/CSS)

## TOC

## :where 和 :is
[:where() - CSS：层叠样式表 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:where)
[:is() - CSS：层叠样式表 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:is)

:is() CSS 伪类函数以选择器列表作为参数，并选择该列表中任意一个选择器可以选择的元素。

:where() CSS 伪类函数接受选择器列表作为它的参数，将会选择所有能被该选择器列表中任何一条规则选中的元素。

```css

/** 给二级列表添加样式 */

:is(ol, ul) :is(ol, ul) ol {

    list-style-type: lower-greek;

    color: chocolate;

}



/* Selects any paragraph inside a header, main

   or footer element that is being hovered */

:where(header, main, footer) p:hover {

  color: red;

  cursor: pointer;

}

```

两者之间的区别在于，:is() 会计入整个选择器的优先级（它采用其最具体参数的优先级），而 :where() 的优先级为 0。这可以通过 :where() 参考页面上的示例来演示。

规范将 :is() 和 :where() 定义为接受容错选择器列表。
1. :is() 伪类可以大大简化您的 CSS 选择器。
2. :is() 不能选择伪元素


## 视觉格式化模型

[视觉格式化模型 - CSS：层叠样式表 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Visual_formatting_model)

- 块（block）：块在文档流中占据独立的区域，块与块之间在垂直方向按照顺序依次堆叠
- 包含块（containing block）：包含其他盒子的块称为包含块
- 盒子（box）：由CSS引擎根据文档中的内容创建，主要用于文档元素的定位、布局、格式化等用途。
- 块级元素（block level element）：元素的display为block、table、list-item时，该元素将成为块级元素。元素是否是块级元素，仅是元素本身的属性，并不直接用于格式化上下文的创建或布局。
- 块级盒子（block level box）：由块级元素生成。一个块级元素至少包含一个块级盒子，但也可能生成多个（例如列表项元素）
- 块盒子（block box）：如果一个块级盒子同时也是一个块容器盒子，则称其为块盒子。除具名块盒子之外，还有一类块盒子是匿名的，这类盒子被称为匿名块盒子（anonymuse block box），匿名盒子无法被css选择器选中。
- 块容器盒子（block containing box）：块容器盒子侧重于当前盒子作为*容器*这一角色，它不参与当前块的布局与定位，描述的仅仅是当前盒子与其后代之间的关系。
- 行内级元素（inline level element）：display为**inline**、**inline-block**、**inline-table**的元素称为行内级元素。与块级元素一样，元素是否是行内级元素仅是元素本身的属性，并不直接用于格式化上下文的创建与布局。
- 行内级盒子（inline-level box）：由行内级元素生成。行内级盒子包含行内盒子和原子行内级盒子两种，区别在于该盒子是否参与行内格式化上下文的创建。
- 行内盒子（inline box）：参与行内格式化上下文创建的行内级盒子称为行内盒子。与块盒子类似，行内盒子也分为具名行内盒子和匿名行内盒子（anonymous inline box）两种。
- 原子行内级盒子（atomic inline-level box）：不参与行内格式化上下文创建的行内级盒子。原子行内级盒子一开始叫做原子行内盒子（atomic inline box），后被修正。原子行内级盒子的内容不会拆分成多行显示。

每个块级盒子都会参与*块格式化上下文*的创建，而每个块级元素都至少会生成一个块级盒子，即主块级盒子（principal block-level box）。
主块级盒子包含由后代元素生成的盒子及其内容，同时它也会参与到*定位方案*。
一个块级盒子可能也是一个块容器盒子。块容器盒子要么只包含其他块级盒子，要么只包含行内盒子并同时创建一个行内格式化上下文。

块级盒子描述了元素与其父元素和兄弟元素之间的行为，而块容器盒子描述了元素跟其后代之间的行为。一个同时是块容器盒子的块级盒子被称为块盒子。

在某些情况下进行视觉格式化时需要添加增补性的盒子，这些盒子不能用CSS选择器选中，因此被称为匿名盒子。匿名盒子所有可继承的CSS属性值都是`inherit`，而所有不可继承的CSS属性值都为`initial`。创建匿名块盒子的情形如下：
1. 块包含行内级盒子和块级盒子时，会在相邻的行内级盒子外创建匿名块盒子。
2. 行内盒子包含一或多个块盒子。此时，包含块盒子的盒子会拆分为两个行内盒子，分别位于块盒子的前后。
```html
<!-- 1 -->
<div class="flex">
I am wrapped in an anonymous box
<p>I am in the paragraph</p>
I am wrapped in an anonymous box.
</div>

<!-- 2 -->
<div class="example">
I am wrapped in an anonymous box
<p>I am in the paragraph</p>
I am wrapped in an anonymous box.
</div>
```



## 定位方案

- 普通流：按照次序依次定位每个盒子
- 浮动：将盒子从普通流中单独拎出来，将其放到盒子的某一边
- 绝对定位：按照绝对位置来定位盒子，其位置根据盒子的包含元素所建立的绝对坐标系来计算，因此绝对定位元素有可能覆盖其他元素。

### 普通流

按照次序依次定位每个盒子。
当CSS的`position`属性为`static`或`relative`，并且`float`为`none`时，其布局方式为普通流。
在块格式化上下文中，盒子在垂直方向依次排列；在行内格式化上下文中，盒子则水平排列。
普通流又有两种情况：
- positon为static时为静态定位，此时每个盒子根据普通流所计算出的确切位置来定位。
- position为relative时为相对定位，此时每个盒子还会根据top、bottom、left、right属性的值在其原本所在的位置上产生指定大小偏移。

### 浮动

浮动盒子会浮动到当前行的开始和尾部位置，这会导致普通流中的文本及其他内容会“流”到浮动盒子边缘处，除非元素通过clear清楚了前面的浮动。
一个盒子的float值不为none，并且其position为static或relative时，该盒子为浮动定位。

行盒子会伸缩以适应浮动盒子的大小。

### 绝对定位

绝对定位中盒子会完全从当前流中移除，并不会在与其有任何联系（仅指位置和定位计算）。

元素的position为absolute或fixed，该元素为绝对定位。

## 应用或脱离流布局

脱离常规流的元素：
- floated items。浮动的元素
- items with `position: absolute` (including `position: fixed` which acts in the same way)。通过设置 position 属性为 absolute 或者 fixed 的元素
- the root element (`html`) 根元素
脱离常规流的元素会创建新的块级格式化上下文环境，其中包含对的所有元素构成了一个小的布局环境，与页面中的其他内容分割开来。而根元素，作为所有页面的容器自身脱离常规流，为整个文档创建一个块级格式化上下文环境。

当移动元素或使元素脱离文档流，为防止折叠，你可能需要对元素内容和严肃周围内容做一些管理，要么清楚浮动，要么保证相对定位不会覆盖其他元素。


## 块格式化上下文

>块格式化上下文（Block Formatting Context，BFC）：是Web页面的可视CSS渲染的一部分，是块级盒子布局过程发生的区域，也是浮动元素与其他元素交互的区域。

格式化上下文包含以下几种类型：
1. 块格式化上下文。根据布局块规则布局子元素
2. 内联格式化上下文。
3. 灵活格式化上下文。将其子元素布局为灵活项（flex items）

下列方式会创建块格式化上下文：
- 根元素（`<html>`）
- 浮动元素（[`float`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/float) 值不为 `none`）
- 绝对定位元素（[`position`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/position) 值为 `absolute` 或 `fixed`）
- 行内块元素（[`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 值为 `inline-block`）
- 表格单元格（[`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 值为 `table-cell`，HTML 表格单元格默认值）
- 表格标题（[`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 值为 `table-caption`，HTML 表格标题默认值）
- 匿名表格单元格元素（[`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 值为 `table`、`table-row`、 `table-row-group`、`table-header-group`、`table-footer-group`（分别是 HTML table、tr、tbody、thead、tfoot 的默认值）或 `inline-table`）
- [`overflow`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/overflow) 值不为 `visible`、`clip` 的块元素
- [`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 值为 `flow-root` 的元素
- [`contain`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/contain) 值为 `layout`、`content` 或 `paint` 的元素
- 弹性元素（[`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 值为 `flex` 或 `inline-flex` 元素的直接子元素），如果它们本身既不是 [flex](https://developer.mozilla.org/zh-CN/docs/Glossary/Flex_Container)、[grid](https://developer.mozilla.org/zh-CN/docs/Glossary/Grid_Container) 也不是 [table](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_table) 容器
- 网格元素（[`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 值为 `grid` 或 `inline-grid` 元素的直接子元素），如果它们本身既不是 [flex](https://developer.mozilla.org/zh-CN/docs/Glossary/Flex_Container)、[grid](https://developer.mozilla.org/zh-CN/docs/Glossary/Grid_Container) 也不是 [table](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_table) 容器
- 多列容器（[`column-count`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/column-count) 或 [`column-width` (en-US)](https://developer.mozilla.org/en-US/docs/Web/CSS/column-width "Currently only available in English (US)") 值不为 `auto`，包括`column-count` 为 `1`）
- `column-span` 值为 `all` 的元素始终会创建一个新的 BFC，即使该元素没有包裹在一个多列容器中 ([规范变更](https://github.com/w3c/csswg-drafts/commit/a8634b96900279916bd6c505fda88dda71d8ec51), [Chrome bug](https://bugs.chromium.org/p/chromium/issues/detail?id=709362))


例子：
```html
<div class="box">
   <div class="float">I am a floated box!</div>
   <p>I am content inside the container.</p>
</div>
```
```css
.box {
   background-color: rgb(224, 206, 247);
   border: 5px solid rebeccapurple;
}
```

![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230705153904.png)

在上面例子中由于float的内容比旁边内容高，所以float的元素贯穿了box；浮动脱离文档流，因此背景和边框仅包含内容不包含浮动。
创建一个新的BFC将包含该浮动，如外部容器元素设置`overflow: auto`或其他不是`overflow: visible`的值。
推荐一个新的方式显式创建BFC：`display: flow-root`，不会产生任何其他潜在的副作用，容器下的所有内容都将参与容器的块格式上下文，并且浮动不会从元素底部弹出。



## 文本断行与包装文本

[Overflowing text | MDN Play](https://developer.mozilla.org/en-US/play?id=qtxraz4LZy72oE60ZNsdlUHKQY0jyA9wIzoBIO6VRfXRP2V274cvwwOEnKqp6SgDkbB2ea0tyAh0b1e5)
[CSS Box Sizing Module Level 3](https://www.w3.org/TR/css-sizing-3/)
[CSS Writing Modes Level 3](https://www.w3.org/TR/css-writing-modes-3/)


```html
<div class="preview">
  <div class="box">Llanfairpwllgwyngyllgogerychwyrndrobwllllantysiliogogogoch</div>
</div>
```
```css
.preview {
  border: 1px solid;
  padding: 16px;
}
.box {
  inline-size: 50px;
  border: 1px solid red;
}
```
overflow默认visible，故文本超出时还能继续看到。
inline-size或width可设置为min-width，宽度设置为最小内容宽度，
如果需要内容宽度适应inline-size，但还要能完整换行展示出来，可配合使用`overflow-wrap:break-word`，这个样式会将超出的单词完整换行（可用`word-break`替代）

单词边界换行方式也可选择增加连字符：`hyphens: manual`

## Shape

[CSS Shapes Module Level 2](https://drafts.csswg.org/css-shapes-2/)


CSS的Shape相关规范定义了浮动图像元素的任意几何形状。
其他类似的规范有：
- [CSS Masking - CSS：层叠样式表 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_masking) 定义部分或完全隐藏视觉部分元素区域
- [CSS Exclusions Module Level 1](https://www.w3.org/TR/css3-exclusions/) 定义了容器盒子内包装上下文的排除区域与内联流内容的环绕（流动）关系


![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230705200647.png)
```css
.float {
	width: 100px;
	height: 100px;
	shape-outside: polygon(10px 10px, 90px 50px, 40px 50px, 90px 90px, 10px 90px);
	shape-margin: 10px;
}
```
