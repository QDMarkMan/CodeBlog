# 一些尺寸相关的API

在前端开发中，一系列的宽度困扰着我们。很多都容易弄混淆，下面我们一次把他们区分开来。

## 写在前面

想要看到懂这篇文章，要自备[CSS盒模型](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Box_Model/Introduction_to_the_CSS_box_model)知识哦。

## clientWidth和clientHeight

`Element.clientWidth/clientHeight` 属性表示元素的**可视区域宽度/高度**，以像素计。该属性**包括内边距，但不包括垂直滚动条（如果有）、边框和外边距**。下面是MDN给的图

![client](./images/client.png 'client')

**只读属性**

注意： 该属性值会被四舍五入为一个整数。如果你需要一个小数值。

计算方式:

`clientWidth/clientHeight = width/height + padding`


## offsetWidth和offsetHeight

`HTMLElement.offsetWidth/offsetHeight`返回一个元素的**布局宽度**。一个典型的`offsetWidth/offsetHeight`是测量包含元素的边框(`border`)、水平线上的内边距(`padding`)、竖直方向滚动条(`scrollbar`)（如果存在的话）、以及CSS设置的宽度(`width/height`)的值。

![offset](./images/offset.png 'offset')

计算方式:

`offsetWidth/offsetHeight = width/height + border + padding + scrollbar`

## scrollHeight和scrollWidth

`Element.scrollHeight/scrollWidth`这个**只读属性**是一个元素内容高度的度量，**包括由于溢出导致的视图中不可见内容**。包括元素的`padding`，但不包括元素的`border`和`margin`.

注意点：

- `scrollHeight/scrollWidth` 的值等于该元素在不使用滚动条的情况下为了适应视口中所用内容所需的最小高度。
- 没有垂直滚动条的情况下，`scrollHeight/scrollWidth`值与元素视图填充所有内容所需要的最小值`clientHeight`相同。
- `scrollHeight/scrollWidth`也包括 `::before` 和 `::after`这样的伪元素。

![scroll](./images/scroll.png 'scroll')

计算方式：

`clientWidth/clientHeight = width/height + padding + ::before + ::after + Invisible part size`


## Element.getBoundingClientRect()

说到尺寸就不得不说说一说她
> Element.getBoundingClientRect()方法返回元素的大小及其相对于视口的位置。

在获取的元素的位置的这一方面，这个`API`是极其的好用。

返回的`DOMRect`格式是这样的。

```js
{
  "x":74,
  "y":245,
  "width":310,
  "height":196,
  "top":245,
  "right":384,
  "bottom":441,
  "left":74
}
```

## Window.getComputedStyle(element, pseudoElt)

`pseudoElt`: 指定一个要匹配的伪元素的字符串。必须对普通元素省略（或`null`）。

这个方法方法返回一个在应用活动样式表并解析这些值可能包含的任何基本计算后报告元素的所有CSS属性的值的对象。 私有的CSS属性值可以通过对象提供的API或通过简单地使用CSS属性名称进行索引来访问。

返回的`style`是一个实时的 [CSSStyleDeclaration(CSS属性键值对集合)](https://developer.mozilla.org/zh-CN/docs/Web/API/CSSStyleDeclaration)只读对象，当元素的样式更改时，它会自动更新本身。

![style](./images/getComputedStyle.png 'style')


## JavaScript窗口属性

其实和上面的都是一样的，不过这里总结一下看的更清晰

- `document.body.clientHeight/clientWidth`: 网页可见区域高/宽
- `document.body.offsetHeight /offsetWidth`: 网页可见区域高/宽(包括边线)
- `document.body.scrollHeight/scrollWidth`: 网页正文全文高/宽
- `document.body.scrollTop/scrollLeft`: 网页被卷去的上/左部分
- `window.screenTop/screenLeft`: 网页正文部分上/左部分
- `window.screen.availHeight/availWidth`: 屏幕可用工作区高/宽部分

# 总结

这些都是平常都会用到的知识，总结出来好区分，省的总是弄混淆。

如果觉得有用得话给个⭐吧🤭