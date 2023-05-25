---
title: responsive-web-design
toc: true
tags:
  - javascript
  - css
date: 2018-12-13 17:57:19
---

自适应一般是设定基准值，宽、高、字体大小都指定为基准值的百分比。当基准值改变时，页面元素、宽高也会按比例变化。

# 自适应宽度

## 不使用绝对宽度

网页宽度默认等于屏幕宽度。所以大部分时候只要不适用绝对宽度即可实现自适应宽度：

```scss
body: {
    width: 100%;
    // or width: auto;
}
```

如果元素是图片，也可以使用 `max-width` 属性，参见[responsive web design: image](https://www.w3schools.com/css/css_rwd_images.asp)

```
img {
  max-width: 100%;
  height: auto;
}
```

## 使用 media

这适用于需要针对不同的屏幕，显示不同的排版。利用 [`@media`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@media) 的 css 规则，可实现根据一个或多个基于设备类型、具体特点和环境的媒体查询来应用样式。

```scss
/* Media query */
@media screen and (min-width: 900px) {
  article {
    padding: 1rem 3rem;
  }
}

/* Nested media query */
@supports (display: flex) {
  @media screen and (min-width: 900px) {
    article {
      display: flex;
    }
  }
}
```

### css 规则

[css 规则](https://developer.mozilla.org/zh-CN/docs/Web/CSS/At-rule) 相当于给 css 增加的函数。原本 css 只是简单的 json 对象，现在希望 css 更多样化、更容易维护，所以增加了 css 规则。

`@media` 这种属于条件规则组，即接收两个参数，第一个参数是个 bool 条件，第二个参数是执行的语句。仅当条件为 true 时，其后的语句才会执行。

# 自适应高度

同样的，要使得高度自适应，也必须指定一个基准值。但不同于宽度（默认是屏幕宽度），高度通常是不固定的，要确定基准值也就麻烦些。

## 父容器的高度可确定

> For the height of a div to be responsive, it must be inside a parent element with a defined height to derive it's relative height from.

如果父容器的高度可确定，则同上，采用百分比即可实现自适应。

```scss
parent: {
    height: 100px;

    child: {
        height: 80%;
    }
}
```

## 根据宽度变化，同比例改变高度

页面宽度的自适应容易实现，可以基于宽度的变化，同比例更改高度。

### 利用 `padding`

参见[responsive height proportional to width](https://stackoverflow.com/questions/11535827/responsive-height-proportional-to-width)，设定 `height` 为 0，`padding` 为百分比

> In all browsers, when padding is specified in %, it's calculated relative to the parent element's width. 

Html:

```html
<div class="content">
    Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum.
</div>

<br/>

<div class="wrapper">
    <div class="content2">
Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat.
    </div>
</div>
```

Css:

```scss
.content {
    width: 100%;
    height: 0;
    padding-bottom: 30%;
    background-color: #7ad;
}
.wrapper {
    position: relative;
    width: 100%;
    padding-bottom: 30%;
    background-color: #7ad;
}
.wrapper .content2 {
    position: absolute;
    width: 100%;
    height: 100%;
    background-color: #7da;
}
```

### 利用 viewport units

新版本的浏览器支持 [viewport units](https://caniuse.com/#feat=viewport-units)，eg.

```scss
width: 100vw;
height: 30vw; /* 30% of width */
```

## 使用 media

同宽度，指定不同 media 情况下，高度显示不同。