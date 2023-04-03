---
title: "SVG Tutorial"
pubDate: "2023-04-03"
slug: "SVG Tutorial"
description: "What is SVG and how to develop SVG"
hero: "/images/svg.png"
tags: ["SVG"]
layout: "../../layouts/BlogPostLayout.astro"
---

## What is SVG
SVG 是一种 XML 语言，类似 XHTML，可以用来绘制矢量图形。

## Simple example
```xml
<svg version="1.1"
     baseProfile="full"
     width="300" height="200"
     xmlns="http://www.w3.org/2000/svg">

  <rect width="100%" height="100%" fill="red" />

  <circle cx="150" cy="100" r="80" fill="green" />

  <text x="150" y="125" font-size="60" text-anchor="middle" fill="white">SVG</text>

</svg>
```

从svg根元素开始：
应舍弃来自 (X)HTML 的 doctype 声明，因为基于 SVG 的 DTD 验证导致的问题比它能解决的问题更多。
</br>
SVG 2 之前，version 属性和 baseProfile 属性用来供其他类型的验证识别 SVG 的版本。SVG 2 不推荐使用 version 和 baseProfile 这两个属性。
</br>
作为 XML 的一种方言，SVG 必须正确的绑定命名空间（在 xmlns 属性中绑定）。

## Grid
对于所有元素，SVG 使用的坐标系统或者说网格系统，和Canvas用的差不多（所有计算机绘图都差不多）。这种坐标系统是：以页面的左上角为 (0,0) 坐标点，坐标以像素为单位，x 轴正方向是向右，y 轴正方向是向下。注意，这和你小时候所教的绘图方式是相反的。但是在 HTML 文档中，元素都是用这种方式定位的。

```
<svg width="200" height="200" viewBox="0 0 100 100">
```
这里定义的画布尺寸是 200*200px。但是，viewBox 属性定义了画布上可以显示的区域：从 (0,0) 点开始，100 宽*100 高的区域。这个 100*100 的区域，会放到 200*200 的画布上显示。于是就形成了放大两倍的效果。

## Basic shapes
SVG 中的基本图形有：矩形、圆形、椭圆、线、折线、多边形、路径。
```xml
<?xml version="1.0" standalone="no"?>
<svg width="200" height="250" version="1.1" xmlns="http://www.w3.org/2000/svg">

  <rect x="10" y="10" width="30" height="30" stroke="black" fill="transparent" stroke-width="5"/>
  <rect x="60" y="10" rx="10" ry="10" width="30" height="30" stroke="black" fill="transparent" stroke-width="5"/>

  <circle cx="25" cy="75" r="20" stroke="red" fill="transparent" stroke-width="5"/>
  <ellipse cx="75" cy="75" rx="20" ry="5" stroke="red" fill="transparent" stroke-width="5"/>

  <line x1="10" x2="50" y1="110" y2="150" stroke="orange" fill="transparent" stroke-width="5"/>
  <polyline points="60 110 65 120 70 115 75 130 80 125 85 140 90 135 95 150 100 145"
      stroke="orange" fill="transparent" stroke-width="5"/>

  <polygon points="50 160 55 180 70 180 60 190 65 205 50 195 35 205 40 190 30 180 45 180"
      stroke="green" fill="transparent" stroke-width="5"/>

  <path d="M20,230 Q40,205 50,230 T90,230" fill="none" stroke="blue" stroke-width="5"/>
</svg>

```

## Path
### 直线
#### 直线命令
- M x y 或 m dx dy
- L x y (or l dx dy)
- H x (or h dx)
- V y (or v dy)
- Z (or z)

```xml
 <path d="M10 10 h 80 v 80 h -80 Z" fill="transparent" stroke="black"/>
```

#### 曲线命令
贝塞尔曲线
</br>
我们从稍微复杂一点的三次贝塞尔曲线 C 入手，三次贝塞尔曲线需要定义一个点和两个控制点，所以用 C 命令创建三次贝塞尔曲线，需要设置三组坐标参数：
```
C x1 y1, x2 y2, x y
(or)
c dx1 dy1, dx2 dy2, dx dy
```
```xml
<svg width="190" height="160" xmlns="http://www.w3.org/2000/svg">

  <path d="M 10 10 C 20 20, 40 20, 50 10" stroke="black" fill="transparent"/>
  <path d="M 70 10 C 70 20, 110 20, 110 10" stroke="black" fill="transparent"/>
  <path d="M 130 10 C 120 20, 180 20, 170 10" stroke="black" fill="transparent"/>
  <path d="M 10 60 C 20 80, 40 80, 50 60" stroke="black" fill="transparent"/>
  <path d="M 70 60 C 70 80, 110 80, 110 60" stroke="black" fill="transparent"/>
  <path d="M 130 60 C 120 80, 180 80, 170 60" stroke="black" fill="transparent"/>
  <path d="M 10 110 C 20 140, 40 140, 50 110" stroke="black" fill="transparent"/>
  <path d="M 70 110 C 70 140, 110 140, 110 110" stroke="black" fill="transparent"/>
  <path d="M 130 110 C 120 140, 180 140, 170 110" stroke="black" fill="transparent"/>

</svg>

```
你可以将若干个贝塞尔曲线连起来，从而创建出一条很长的平滑曲线。通常情况下，一个点某一侧的控制点是它另一侧的控制点的对称（以保持斜率不变）。这样，你可以使用一个简写的贝塞尔曲线命令 S，如下所示：
```
S x2 y2, x y
(or)
s dx2 dy2, dx dy
```
```xml
<svg width="190" height="160" xmlns="http://www.w3.org/2000/svg">
  <path d="M 10 80 C 40 10, 65 10, 95 80 S 150 150, 180 80" stroke="black" fill="transparent"/>
</svg>
```

另一种可用的贝塞尔曲线是二次贝塞尔曲线 Q，它比三次贝塞尔曲线简单，只需要一个控制点，用来确定起点和终点的曲线斜率。因此它需要两组参数，控制点和终点坐标。
`Q x1 y1, x y
(or)
q dx1 dy1, dx dy`

```xml
<svg width="190" height="160" xmlns="http://www.w3.org/2000/svg">
  <path d="M 10 80 Q 95 10 180 80" stroke="black" fill="transparent"/>
</svg>
```

就像三次贝塞尔曲线有一个 S 命令，二次贝塞尔曲线有一个差不多的 T 命令，可以通过更简短的参数，延长二次贝塞尔曲线。
`T x y
(or)
t dx dy`

```xml
<svg width="190" height="160" xmlns="http://www.w3.org/2000/svg">
  <path d="M 10 80 Q 52.5 10, 95 80 T 180 80" stroke="black" fill="transparent"/>
</svg>
```

### 弧形
弧形命令 A
`A rx ry x-axis-rotation large-arc-flag sweep-flag x y
(or)
a rx ry x-axis-rotation large-arc-flag sweep-flag dx dy`

## Fill 和 Stroke 属性
### Fill
- fill: none | <color> | <paint> | inherit
- fill-opacity: <opacity-value> | inherit
- fill-rule: nonzero | evenodd | inherit
- fill: url(#gradient)

### Stroke
- stroke: none | <color> | <paint> | inherit
- stroke-dasharray: none | <dasharray> | inherit
- stroke-dashoffset: <dashoffset> | inherit
- stroke-linecap: butt | round | square | inherit
- stroke-linejoin: miter | round | bevel | inherit
- stroke-miterlimit: <miterlimit> | inherit
- stroke-opacity: <opacity-value> | inherit
- stroke-width: <length> | inherit
- stroke: url(#gradient)

### Use CSS
除了定义对象的属性外，你也可以通过 CSS 来样式化Fill和Stroke。

## Gradients
### Linear Gradients
```xml
<svg width="190" height="160" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <linearGradient id="MyGradient">
      <stop offset="5%" stop-color="#ff0000"/>
      <stop offset="95%" stop-color="#0000ff"/>
    </linearGradient>
  </defs>
  <rect x="10" y="10" width="150" height="100" fill="url(#MyGradient)"/>
</svg>
```

### Radial Gradients
Basic Example
```xml
<svg width="190" height="160" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <radialGradient id="MyGradient">
      <stop offset="5%" stop-color="#ff0000"/>
      <stop offset="95%" stop-color="#0000ff"/>
    </radialGradient>
  </defs>
  <ellipse cx="100" cy="70" rx="85" ry="55" fill="url(#MyGradient)"/>
</svg>
```

Center and focal point
```xml
<?xml version="1.0" standalone="no"?>

<svg width="120" height="120" version="1.1" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <radialGradient id="Gradient" cx="0.5" cy="0.5" r="0.5" fx="0.25" fy="0.25">
      <stop offset="0%" stop-color="red" />
      <stop offset="100%" stop-color="blue" />
    </radialGradient>
  </defs>

  <rect
    x="10"
    y="10"
    rx="15"
    ry="15"
    width="100"
    height="100"
    fill="url(#Gradient)"
    stroke="black"
    stroke-width="2" />

  <circle
    cx="60"
    cy="60"
    r="50"
    fill="transparent"
    stroke="white"
    stroke-width="2" />
  <circle cx="35" cy="35" r="2" fill="white" stroke="white" />
  <circle cx="60" cy="60" r="2" fill="white" stroke="white" />
  <text x="38" y="40" fill="white" font-family="sans-serif" font-size="10pt">
    (fx,fy)
  </text>
  <text x="63" y="63" fill="white" font-family="sans-serif" font-size="10pt">
    (cx,cy)
  </text>
</svg>
```

spreadMethod
```xml
<?xml version="1.0" standalone="no"?>

<svg width="220" height="220" version="1.1" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <radialGradient
      id="GradientPad"
      cx="0.5"
      cy="0.5"
      r="0.4"
      fx="0.75"
      fy="0.75"
      spreadMethod="pad">
      <stop offset="0%" stop-color="red" />
      <stop offset="100%" stop-color="blue" />
    </radialGradient>
    <radialGradient
      id="GradientRepeat"
      cx="0.5"
      cy="0.5"
      r="0.4"
      fx="0.75"
      fy="0.75"
      spreadMethod="repeat">
      <stop offset="0%" stop-color="red" />
      <stop offset="100%" stop-color="blue" />
    </radialGradient>
    <radialGradient
      id="GradientReflect"
      cx="0.5"
      cy="0.5"
      r="0.4"
      fx="0.75"
      fy="0.75"
      spreadMethod="reflect">
      <stop offset="0%" stop-color="red" />
      <stop offset="100%" stop-color="blue" />
    </radialGradient>
  </defs>

  <rect
    x="10"
    y="10"
    rx="15"
    ry="15"
    width="100"
    height="100"
    fill="url(#GradientPad)" />
  <rect
    x="10"
    y="120"
    rx="15"
    ry="15"
    width="100"
    height="100"
    fill="url(#GradientRepeat)" />
  <rect
    x="120"
    y="120"
    rx="15"
    ry="15"
    width="100"
    height="100"
    fill="url(#GradientReflect)" />

  <text x="15" y="30" fill="white" font-family="sans-serif" font-size="12pt">
    Pad
  </text>
  <text x="15" y="140" fill="white" font-family="sans-serif" font-size="12pt">
    Repeat
  </text>
  <text x="125" y="140" fill="white" font-family="sans-serif" font-size="12pt">
    Reflect
  </text>
</svg>
```

## patterns(图案)

## Texts
### Basic example
```xml
<svg width="190" height="160" xmlns="http://www.w3.org/2000/svg">
  <text x="10" y="20" fill="red">I love SVG!</text>
</svg>
```

### tspan
该元素用来标记大块文本的子部分，它必须是一个text元素或别的tspan元素的子元素。一个典型的用法是把句子中的一个词变成粗体红色。

```xml
<text>
  <tspan font-weight="bold" fill="red">This is bold and red</tspan>
</text>
```

### tref
```xml
<svg width="190" height="160" xmlns="http://www.w3.org/2000/svg">
    <text id="example">This is an example text.</text>
    <text>
        <tref xlink:href="#example" />
    </text>
</svg>
```

### textPath
该元素利用它的xlink:href属性取得一个任意路径，把字符对齐到路径，于是字体会环绕路径、顺着路径走：
```xml
<svg width="190" height="160" xmlns="http://www.w3.org/2000/svg">
    <path id="my_path" d="M 20,20 C 40,40 80,40 100,20" fill="transparent" />
    <text>
      <textPath xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#my_path">
        This text follows a curve.
      </textPath>
    </text>
</svg>
```

## Basic Transformations
- translate
- scale
- rotate
- skewX
- skewY
- matrix

## Clipping and Masking
### Clipping
```xml
<svg width="190" height="160" xmlns="http://www.w3.org/2000/svg">
    <defs>
        <clipPath id="myClip">
            <rect x="10" y="10" width="100" height="100" />
        </clipPath>
    </defs>
    <rect x="0" y="0" width="190" height="160" fill="red" />
    <rect x="0" y="0" width="190" height="160" fill="blue" clip-path="url(#myClip)" />
</svg>
```

### Masking
```xml
<svg width="190" height="160" xmlns="http://www.w3.org/2000/svg">
    <defs>
        <mask id="myMask">
            <rect x="10" y="10" width="100" height="100" fill="white" />
            <circle cx="60" cy="60" r="50" fill="black" />
        </mask>
    </defs>
    <rect x="0" y="0" width="190" height="160" fill="red" />
    <rect x="0" y="0" width="190" height="160" fill="blue" mask="url(#myMask)" />
</svg>
```

## Transparency with opacity
```xml
<svg width="190" height="160" xmlns="http://www.w3.org/2000/svg">
    <rect x="0" y="0" width="190" height="160" fill="red" />
    <rect x="0" y="0" width="190" height="160" fill="blue" opacity="0.5" />
</svg>
```

## Other content in SVG
### Embedding raster images
```xml
<svg width="190" height="160" xmlns="http://www.w3.org/2000/svg">
    <image x="10" y="10" width="100" height="100" xlink:href="http://www.w3.org/Icons/SVG/svg-logo-v.svg" />
</svg>
```

### Embedding arbitrary XML
```xml
<svg viewBox="0 0 200 200" xmlns="http://www.w3.org/2000/svg">
  <style>
    div {
      color: white;
      font: 18px serif;
      height: 100%;
      overflow: auto;
    }
  </style>

  <polygon points="5,5 195,10 185,185 10,195" />

  <!-- Common use case: embed HTML text into SVG -->
  <foreignObject x="20" y="20" width="160" height="160">
    <!--
      In the context of SVG embedded in an HTML document, the XHTML
      namespace could be omitted, but it is mandatory in the
      context of an SVG document
    -->
    <div xmlns="http://www.w3.org/1999/xhtml">
      Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed mollis mollis
      mi ut ultricies. Nullam magna ipsum, porta vel dui convallis, rutrum
      imperdiet eros. Aliquam erat volutpat.
    </div>
  </foreignObject>
</svg>
```

## Filter effects
- feBlend
- feColorMatrix
- feComponentTransfer
- feComposite
- feConvolveMatrix
- feDiffuseLighting
- feDisplacementMap
- feFlood
- feGaussianBlur
- feImage
- feMerge
- feMorphology
- feOffset
- feSpecularLighting
- feTile
- feTurbulence
- feDistantLight
- fePointLight
- feSpotLight
- feFuncR
- ...

## SVG fonts

## SVG image element

## Additional Resources

- [MDN](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Tutorial)
