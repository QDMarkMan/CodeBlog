# 快速上手的CSS片段
CSS这东西说难也不难，说简单也复杂的很。其实还是主要靠一个积累。这个repo主要用来积累常用`CSS片段`(<small>动画居多</small>)
## 布局
- 等宽高比的盒子
  给定可变宽度的元素，它将确保其高度以响应的方式保持成比例(即，其宽度与高度的比率保持恒定)。

  ```html
    <div class="constant-width-to-height-ratio"></div>
  ```

  ```less
  // 等宽高比
  .constant-width-to-height-ratio{
    background-color: #333;
    width: 50%;
    &::before {
      content: '';
      padding-top: 100%; // 这个百分比相对的是当前元素宽度的百分比。 因此表示元素的高度始终为100% 的宽度，创建一个响应正方形。
      float: left;
    }
    &::after {
      content: '';
      display: block;
      clear: both;
    }
  }
  ```

## 动画

动画长一定要理解的就是`animate` 和 `transition`这两个属性

`transition: property(过渡的元素) duration(过渡时长) timing-function(过渡曲线) delay(延迟时间);`

注意： **当 property中 height 的值是 length，百分比或 calc() 时支持 CSS3 过渡。**

`animation: name(动画名称) duration(单次动画持续时长) timing-function(动画过渡曲线) delay(延迟执行时间) iteration-count(动画播放次数：n|infinite) direction(是否应该轮流反向播放动画：normal|alternate) fill-mode(规定填充模式： none | forwards | backwards | both);`

- `CSS`弹跳加载动画。
```html
<div class="bouncing-loader">
  <div class="loader"></div>
  <div class="loader"></div>
  <div class="loader"></div>
</div>
```
```less
// 跳跃加载动画
.bouncing-loader{
  display: flex;
  justify-content: center;
  .loader{ 
    width: 1rem;
    height: 1rem;
    margin: 3rem .2rem;
    background-color: #8385aa;
    border-radius: 50%;
    animation: bouncing .6s infinite alternate; 
    &:nth-child(2) {
      animation-delay: .2s;
    }
    &:nth-child(3) {
      animation-delay: .4s;
    }
  }
}
// 动画 利用上下的移动和颜色的过渡产生视觉差。
@keyframes bouncing {
  from {
    opacity: 1;
    transform: translateY(0)
  }
  to {
    opacity: 0.1;
    transform: translateY(-1rem)
  }
}
```

## 其他
