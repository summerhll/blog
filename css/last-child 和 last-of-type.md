# 一. :last-child
有时候会遇到使用last-child，但是样式无效的问题。出现这种问题主要是页面布局不对。

el:last-child 的匹配规则是：

    1.查找 el 选择器匹配元素的所有同级元素（siblings）
    2.在同级元素中查找最后一个元素
    3.检验最后一个元素是否与选择器 el 匹配。
    
 
错误示例如下： 
```html
<div class = "content">
<div class = "item">1</div>
<div class = "item">2</div>
<div class = "item">3</div>
<div class = "show-more">查看更多</div>
</div>
```
```css
.item:last-child {
    margin-bottom: 16px;
}
```
上面的样式不起作用，因为.item同级的最后一个元素是show-more，它的class不符合条件，所以样式不会生效。

修改布局如下，则上面的样式生效：
```html
<div class = "content">
<div class ="list">
<div class = "item">1</div>
<div class = "item">2</div>
<div class = "item">3</div>
</div>
<div class = "show-more">查看更多</div>
</div>
```

二. :last-of-type
el:last-of-type 的匹配规则是：

     1.查找 与el 选择器匹配元素“相同标签”的所有同级元素（siblings）。
     2.在上述所找到的元素中查找最后一个元素。
     3.检验最后一个元素是否与选择器 el 匹配。

错误示例：
```html
<div class = "ul">
    <div class = "item">1</div>
    <div class = "item">2</div>
    <div class = "item">3</div>
    <div class = "show-more">查看更多</div>
</div>
```
```css
.item:last-child {
    margin-bottom: 16px;
}
```
上述样式不会生效，因为找到的元素是show-more,不符合要求。

正确写法：修改show-more的标签，让它不是div，则样式可以生效
```html
<div class = "ul">
    <div class = "item">1</div>
    <div class = "item">2</div>
    <div class = "item">3</div>
    <p class = "show-more">查看更多</p>
</div>
```

 综上：last-of-type和last-child的主要区别在于: last-of-type选择器还需要考虑el选择器元素的标签。
