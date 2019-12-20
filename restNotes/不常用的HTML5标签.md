---
title: 不常用的HTML5标签
date: 2019-01-28 16:24:04
tags: html
---

## base
### 定义及用法
> base标签为页面上的所有链接规定默认地址或默认目标。通常情况下，浏览器会从当前文档的 URL 中提取相应的元素来填写相对 URL 中的空白。使用base标签可以改变这一点。浏览器随后将不再使用当前文档的 URL，而使用指定的基本 URL 来解析所有的相对 URL。

### 属性
href
target

### 其他
> 一个文档的基本URL, 可以一个脚本使用 document.baseURI查询。

> 如果指定了多个<base>元素，只会使用第一个href和target值，其余都会被忽略


### Example
```html
// href
<base href="http://www.example.com/"/>
// target
<base target="_blank" />
// both href and target
<base target="_blank" href="http://www.example.com/"/>
```

## figure && figcaption
### 定义及用法
> figure标签规定独立的流内容（图像、图表、照片、代码等等）。figure元素的内容应该与主内容相关，但如果被删除，则不应对文档流产生影响。

> 可以通过在figure元素中插入一个figcaption来为其关联一个标题(作为它的第一个或者最后一个子元素)

### Example
```html
// just a figure
<figure>
    <img src="http://www.example.com/">
</figure>
// figure with figcaption
<figure>
    <img src="http://www.example.com/">
    <figcaption>Logo</figcaption>
</figure>
```

## abbr
### 定义及用法
> 代表缩写，并可选择提供一个完整的描述

### 属性
title: 完整描述

### 其他
所有浏览器默认把此元素显示为行内元素，但是默认样式有差别
- 如IE，和span元素完全一样
- 如firefox，给内容添加一条点状下划线
- 有些不仅添加点状下划线，还把内容添加small caps，为了避免这个样式，可以在CSS中添加font-variant: none

### Example
```html
<abbr title="Laugh Out Loud">LOL</abbr>
```

## cite
### 定义及用法
> 用来描述一个引创造性劳动的引用，并且必须包括标题或作者或工作的URL。根据与引用元数据相关的上下文适当约定，引用可以是缩写形式。

### 其他
> W3C规范表明，对cite元素中包含的创意作品的引用可能包含作品作者的姓名。然而，对于WHATWG规范正好相反：一个人的名字必须永远不会被包含

可以用来标记在blockquote/q标签中包含的引用材料的出处

浏览器默认cite的样式为斜体，可以通过font-style来修改

### Example
```html
<p>More information can be found in <cite>[ISO-0000]</cite>.</p>

<p><q>This is correct, said Hillary.</q> is a quote from the 
   popular daytime TV drama <cite>When Ian became Hillary</cite>.</p>
```


## ruby系列
### 定义及用法
> 被用来展示东亚文字注音或字符注释。

rp: 用于为那些不能使用ruby元素展示ruby注解的浏览器，提供随后的圆括号
rb: 作为一个ruby元素的孩子,分隔注释的文字
rt: 包含字符的发音, rtc: 包含文字的语义注解

### Example
```html
<ruby>
  汉 <rt>Hàn</rt>
  字 <rt>Zì</rt>
</ruby>

<ruby>
  明日 <rp>(</rp><rt>Ashita</rt><rp>)</rp>
</ruby>

<ruby>
  <rb>汉</rb>字
  <rp>(</rp><rt>han</rt>zi<rp>)</rp>
</ruby>
```

## sub
### 定义及用法
> 定义了一个文本区域，出于排版的原因，与主要的文本相比，应该展示得更低并且更小。
> 这个元素应该只用于排版目的，也就是改变文本的位置会改变含义，例如在数学中（t2，也可以考虑使用 MathML 公式）或者化学符号（ H2O）。
> 这个元素不能用于样式上的目的，比如产品名称 LaTeX 的样式，这时应该使用 CSS样式： vertical-align 属性的 sub 值能实现相同效果。

### Example
```html
<p>The chemical formula of water: H<sub>2</sub>O</p>
```

## sup
与sub相反，上标
