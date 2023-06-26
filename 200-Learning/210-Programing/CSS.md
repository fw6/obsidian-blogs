#CSS #Programing #FE

>CSS值得记录的特性

[Learn CSS | web.dev](https://web.dev/learn/css/)
[CSS - 学习 Web 开发 | MDN](https://developer.mozilla.org/zh-CN/docs/Learn/CSS)

## **:where 和 :is**
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


**视觉格式化模型**
[视觉格式化模型 - CSS：层叠样式表 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Visual_formatting_model)

- 块（block）：块在文档流中占据独立的区域，块与块之间在垂直方向按照顺序依次堆叠
- 包含块（containing block）：包含其他盒子的块称为包含块
- 盒子（box）：由CSS引擎根据文档中的内容创建，主要用于文档元素的定位、布局、格式化等用途。
- 块级元素（block level element）：元素的display为block、table、list-item时，该元素将成为块级元素。元素是否是块级元素，仅是元素本身的属性，并不直接用于格式化上下文的创建或布局。
- 块级盒子（block level box）：
- 匿名盒子（anonymous boxes）：
- 块容器盒子（block containing box）：


