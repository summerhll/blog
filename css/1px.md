# 一. 产生原因
在设备像素比大于1的屏幕上，我们写的1px实际上是被多个物理像素渲染，这就会出现1px在有些屏幕上看起来很粗的现象。

0.5px: iOS 8+系统支持，安卓系统不支持。

# 二.解决办法
## 1.伪元素+ transform+@media（自己常用方法）
```css
//移动端1px
.min-device-pixel-ratio(@scale2, @scale3) {
    @media screen and (min-device-pixel-ratio: 2),
    (-webkit-min-device-pixel-ratio: 2) {
        transform: @scale2;
    }

    @media screen and (min-device-pixel-ratio: 3),
    (-webkit-min-device-pixel-ratio: 3) {
        transform: @scale3;
    }
}

.border-1px(@color: black, @radius: 2PX, @style: solid) {
    position: relative;
    &::before {
        content: "";
        pointer-events: none;
        display: block;
        position: absolute;
        left: 0;
        top: 0;
        transform-origin: 0 0;
        border: 1PX @style @color;
        border-radius: @radius;
        box-sizing: border-box;
        width: 100%;
        height: 100%;

        @media screen and (min-device-pixel-ratio: 2),
        (-webkit-min-device-pixel-ratio: 2) {
            width: 200%;
            height: 200%;
            border-radius: @radius * 2;
            transform: scale(.5);
        }

        @media screen and (min-device-pixel-ratio: 3),
        (-webkit-min-device-pixel-ratio: 3) {
            width: 300%;
            height: 300%;
            border-radius: @radius * 3;
            transform: scale(.33);
        }
    }
}

.border-top-1px(@color: black, @style: solid) {
    position: relative;
    &::before {
        content: "";
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        border-top: 1Px @style @color;
        transform-origin: 0 0;
        .min-device-pixel-ratio(scaleY(.5), scaleY(.33));
    }
}

.border-bottom-1px(@color: black, @style: solid) {
    position: relative;

    &::after {
        content: "";
        position: absolute;
        left: 0;
        bottom: 0;
        width: 100%;
        border-bottom: 1Px @style @color;
        transform-origin: 0 0;
        .min-device-pixel-ratio(scaleY(.5), scaleY(.33));
    }
}

.border-left-1px(@color: black, @style: solid) {
    position: relative;
    &::before {
        content: "";
        position: absolute;
        left: 0;
        top: 0;
        height: 100%;
        border-left: 1Px @style @color;
        transform-origin: 0 0;
        .min-device-pixel-ratio(scaleX(.5), scaleX(.33));
    }
}

.border-right-1px(@color: black, @style: solid) {
    position: relative;
    &::before {
        content: "";
        position: absolute;
        right: 0;
        top: 0;
        height: 100%;
        border-right: 1Px @style @color;
        transform-origin: 0 0;
        .min-device-pixel-ratio(scaleX(.5), scaleX(.33));
    }
}

```
## 2.设置viewport的scale值, flexible.js使用的方法
```html
  <html>
  <head>
      <title>1px question</title>
      <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
      <meta name="viewport" id="WebViewport" content="initial-scale=1,
       maximum-scale=1, minimum-scale=1, user-scalable=no">        
      <style>
          html {
              font-size: 1px;
          }            
          * {
              padding: 0;
              margin: 0;
          }
          .top_b {
              border-bottom: 1px solid #E5E5E5;
          }

          .a,.b {
             box-sizing: border-box;
              margin-top: 1rem;
              padding: 1rem;                
              font-size: 1.4rem;
          }

          .a {
              width: 100%;
          }

          .b {
              background: #f5f5f5;
              width: 100%;
          }
      </style>
      <script>
          var viewport = document.querySelector("meta[name=viewport]");
          //下面是根据设备像素设置viewport
          if (window.devicePixelRatio == 1) {
              viewport.setAttribute('content', 'width=device-width,initial-scale=1,
               maximum-scale=1, minimum-scale=1, user-scalable=no');
          }
          if (window.devicePixelRatio == 2) {
              viewport.setAttribute('content', 'width=device-width,initial-scale=0.5,
               maximum-scale=0.5, minimum-scale=0.5, user-scalable=no');
          }
          if (window.devicePixelRatio == 3) {
              viewport.setAttribute('content', 'width=device-width,
              initial-scale=0.3333333333333333, maximum-scale=0.3333333333333333,
               minimum-scale=0.3333333333333333, user-scalable=no');
          }
          var docEl = document.documentElement;
          var fontsize = 32* (docEl.clientWidth / 750) + 'px';
          docEl.style.fontSize = fontsize;
      </script>
  </head>
  <body>
      <div class="top_b a">下面的底边宽度是虚拟1像素的</div>
      <div class="b">上面的边框宽度是虚拟1像素的</div>
  </body>
</html>     

```

## 3. postcss-write-svg（目前比较常用的方法 新方法）
这种方案本质上border并没有变细，但是boder被一分为二，靠内侧的是透明的。
svg只能画出特定的形状，所以无法实现圆角边框。而伪类元素方案可以。

编写代码：
```css
@svg 1px-border {
    height: 2px;
    @rect {
      fill: var(--color, black);
      width: 100%;
      height: 50%;
    }
}
.example {
    border: 1px solid transparent;
    border-image: svg(1px-border param(--color #00b1ff)) 2 2 stretch;
 }
```

这样PostCSS会自动帮你把CSS编译出来
```css
.example {
    border: 1px solid transparent;
    border-image: url("data:image/svg+xml;charset=utf-8,%3Csvg xmlns='http://www.w3.org/2000/svg' height='2px'%3E%3Crect fill='%2300b1ff' width='100%25' height='50%25'/%3E%3C/svg%3E")
          2 2 stretch;
}
```

这个方案简单易用。

## 4. border-image
```css
.border-image-1px {
    border-bottom: 1px solid #666;
} 
 
@media only screen and (-webkit-min-device-pixel-ratio: 2) {
    .border-image-1px {
        border-bottom: none;
        border-width: 0 0 1px 0;
        border-image: url(../img/linenew.png) 0 0 2 0 stretch;
    }
}
```


优点：可以设置单条,多条边框，没有性能瓶颈的问题
缺点：修改颜色麻烦, 需要替换图片；圆角需要特殊处理，并且边缘会模糊

## 5.background-image
```css
.background-image-1px {
  background: url(../img/line.png) repeat-x left bottom;
  background-size: 100% 1px;
}
```

优点：可以设置单条,多条边框，没有性能瓶颈的问题。
缺点：修改颜色麻烦, 需要替换图片；圆角需要特殊处理，并且边缘会模糊。

## 6.box-shadow 
```css
div{
    -webkit-box-shadow:0 1px 1px -1px rgba(0, 0, 0, 0.5);
}
```

优点：圆角不是问题，缺点是颜色不好控制。
