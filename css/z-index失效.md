在工作中遇到这样一个问题：设置一个div z-index 很大，但是仍然被其他元素遮挡。

原因如下：
# 一.父级元素溢出隐藏或者不显示
如果父元素设置了 overflow:hidden /display:none等，那么子元素如果在父元素外部绝对定位，那么调节子元素 z-index 可能不会显示。
```css
.father {
  display: none;
  opacity: 0;
  overflow: hidden; (auto)
  position: relative;
}
.son {
  position: absolute;
  top: 0;
  right: 0;
  z-index: 100;
}
```
解决方案：取消父元素的上述属性。

# 二. 父级元素层级低
```html
<div style="z-index: 1">
  <div style="z-index: 10">son</div>
</div>
<div style="z-index: 2"></div>
```
解决方法：查看父元素的 z-index 并修改

# 三. 没有设置定位
```html
<div style="position:'absolute'; z-index: 10"></div>
```
解决方法：设置定位

# 四.总结
1. z-index只有在设置了position为relative,absolute,fixed时才会有效。
2. z-index的“从父原则”。当你发现把z-index设的多大都没用时，看看其父元素是否设置了有效的z-index，当然别忘了先看看自身是否设置了position。或者查看父级元素是否被隐藏。
