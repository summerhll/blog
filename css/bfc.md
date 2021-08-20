# 一. 概念
BFC是块级格式化上下文。

# 二. 原理
* BFC是一个独立容器，不影响外面也不被外面影响。
* 属于同一个BFC区域的相邻元素垂直方向边距重叠。
* BFC区域不会跟浮动元素重叠
* BFC容器计算高度时会计算浮动元素（子元素）的高度（即清除浮动后父元素有高度）


# 三. 如何创建BFC
* html根元素就是一个BFC
* 浮动元素：float不为none的元素
* 定位元素：position 为absolute、fixed的元素
* display为flex、 inline-block
* overflow不为visible（hidden, auto,scroll）

# 四. BFC的应用场景
* 两栏布局：左边固定，右边自适应
``` html
<div class="parent">
    <div class="left">
        <p>left</p>
    </div>
    <div class="right">
        <p>right</p>
        <p>right</p>
    </div>
</div>

.left{
    float: left;
    width: 100px;
    margin-right: 20px;    //形成20px的间隔
}
.right{
    overflow: hidden; //通过设置overflow: hidden来触发BFC特性
}
``` 

* 清除浮动
``` html
<div style="border: 1px solid #000;overflow: hidden">
    <div style="width: 100px;height: 100px;background: #eee;float: left;"></div>
</div>
```

* 解决上下边距重叠问题
``` html
<div class="container">
    <p></p>
</div>
<div class="container">
    <p></p>
</div>

.container {
    overflow: hidden; //这个必须有，不加这句还是会产生高度塌陷
}
p {
    width: 100px;
    height: 100px;
    background: lightblue;
    margin: 100px;
}
```
