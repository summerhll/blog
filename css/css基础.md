# 一. position 取值
   - static 默认值。没有定位，元素出现在正常的流中。
   - relative：相对定位元素
   
    1.元素先放置在未添加定位时的位置，再在不改变页面布局的前提下调整元素位置（因此会在此元素未添加定位时所在位置留下空白）
   
    
   - absolute：绝对定位元素

    1.元素会被移出正常文档流，并不为元素预留空间
    2.通过指定元素相对于最近的非 static 定位祖先元素的偏移，来确定元素位置

   - fixed：

    1.元素会被移出正常文档流，并不为元素预留空间
    2.通过指定元素相对于屏幕视口（viewport）的位置来指定元素位置
    
   - sticky： 粘性
     当元素在屏幕内，表现为relative，就要滚出显示器屏幕的时候，表现为fixed。
     
     注意：
     
     1.父级元素不能有任何overflow:visible以外的overflow设置，否则没有粘滞效果。因为改变了滚动容器（即使没有出现滚动条）。因此，如果你的position:sticky无效，
     看看是不是某一个祖先元素设置了overflow:hidden，移除之即可。
     2.父级元素设置和粘性定位元素等高的固定的height高度值，或者高度计算值和粘性定位元素高度一样，也没有粘滞效果。
     3.同一个父容器中的sticky元素，如果定位值相等，则会重叠；如果属于不同父元素，且这些父元素正好紧密相连，则会鸠占鹊巢，挤开原来的元素，形成依次占位的效果。
     4.sticky定位，不仅可以设置top，基于滚动容器上边缘定位；还可以设置bottom，也就是相对底部粘滞。如果是水平滚动，也可以设置left和right值。


# 二. 选择器优先级
   !important>行内样式>ID选择器 > 类选择器 | 属性选择器 | 伪类选择器 > 标签选择器｜伪元素选择器> 通配符
   
# 三. display:none VS visibility:hidden
  display: none: 隐藏对应的元素，在文档布局中不再给它分配空间.(display改变引起回流)
  visibility: hidden: 隐藏对应的元素，但在文档布局中仍保留原来的空间。 (visibility改变引起重绘)
  
# 四. CSS 中 link 和@import 的区别是
1. link 属于 HTML 标签，而 @import 是 CSS 提供的;
2. 页面被加载的时，link 会同时被加载，而 @import 引用的 CSS 会等到页面被加载完再加载;
3. import 只在 IE5 以上才能识别，而 link 是 HTML 标签，无兼容问题;
4. link 方式的样式的权重 高于 @import 的权重

# 五 盒模型
CSS盒模型：标准模型 + IE模型（怪异模型）

标准盒模型：盒子总宽度/高度 = width/height + padding + border + margin。（ 即 width/height 只是内容高度，不包含 padding 和 border 值 ）

IE盒子模型：盒子总宽度/高度 = width/height + margin = (内容区宽度/高度 + padding + border) + margin。（ 即 width/height 包含了 padding 和 border 值 ）

标准：box-sizing: content-box; ( 浏览器默认设置 )

IE： box-sizing: border-box;
