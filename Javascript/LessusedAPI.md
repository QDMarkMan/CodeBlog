# Less used and interesting API

> 一些平常比较少用但是很有用的API

## IntersectionObserver API

> `**IntersectionObserver**`**接口** (从属于[Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)) 提供了一种异步观察目标元素与其祖先元素或顶级文档视窗([viewport](https://developer.mozilla.org/zh-CN/docs/Glossary/Viewport))交叉状态的方法。祖先元素与视窗([viewport](https://developer.mozilla.org/zh-CN/docs/Glossary/Viewport))被称为**根(root)。**  --MDN

[Intersection Observer API 文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Intersection_Observer_API)

### 描述

在以往我们需要判断元素的可见性的时候，主流的方式是坚挺`scroll`事件后，调用目标元素的`getBoundingClientRect()`方法，得到对应于视口左上角的坐标， 再判断是否再视口之内，这种方法确定很明显，`scroll`事件密集触发，计算量很大。很容易造成性能问题。`IntersectionObserver `可以"自动"观察元素是否可见，由于可见的本质是**目标元素与可视区域产生一个交叉区**， 所以这个API叫做**"交叉观察器"**

### 用法

创建一个新的`IntersectionObserver`对象，当其监听到目标元素的可见部分穿过了一个或多个**阈(thresholds)**时，会执行指定的回调函数

#### 构造方法

`new IntersectionObserver(callback, option = {});`

- `callback(entries: Array<IntersectionObserverEntry>)`:  可见变化时的回调函数
- `option: Obejct`:  配置对象（可选）

#### 实例属性和方法

属性

- `IntersectionObserver.root` **只读**:  监听对象的具体祖先元素([`element`](https://developer.mozilla.org/zh-CN/docs/Web/API/Element))。如果未传入值或值为`null`，则默认使用顶级文档的视窗。
- `IntersectionObserver.rootMargin` **只读**:  计算交叉时添加到root中的边界盒的举行偏移量， 这个值可以有效的缩小或者扩大根的判定范围。
- `IntersectionObserver.thresholds` **只读**:  一个包含阈值的列表, 按照升序排序，列表中的每个阈值都是监听对象的交叉区域与边界区域的比率。当监听对象的任何阈值被越过时，都会生成一个通知(Notification)。如果构造器未传入值, 则默认值为0。

方法

- ``IntersectionObserver.observe()`：开始监听目标元素。
- ``IntersectionObserver.disconnect()`：停止监听工作。
- ``IntersectionObserver.tabkeRecords()`：返回所有观察目标的[`IntersectionObserverEntry`](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserverEntry)对象数组。
- ``IntersectionObserver.unObserver()`：停止监听。

```js
// 创建观察器实例
const io = new IntersectionObserver(callback, option);
// 开始观察 多次调用observe方法可以观察多个节点
io.observe(document.getElementById('example'));
io.observe(document.getElementById('exampleA'));
// 停止观察
io.unobserve(element)
// 关闭观察器
io.disconnect();
```



#### IntersectionObserverEntry 对象

IntersectionObserverEntry对象用于提供目标元素的信息,  一共有6个属性

- `time`：可见性发生变化的时间，是一个高精度时间戳，单位为毫秒
- `target`：被观察的目标元素，是一个 DOM 节点对象
- `rootBounds`：根元素的矩形区域的信息，`getBoundingClientRect()`方法的返回值，如果没有根元素（即直接相对于视口滚动），则返回`null`
- `boundingClientRect`：目标元素的矩形区域的信息
- `intersectionRect`：目标元素与视口（或根元素）的交叉区域的信息   🚩
- `intersectionRatio`：目标元素的可见比例，即`intersectionRect`占`boundingClientRect`的比例，完全可见时为`1`，完全不可见时小于等于`0`  🚩

## MutationObserver

[MutationObserver文档](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver)



