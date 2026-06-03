---
title: '设计和编写front end component'
description: 'Lorem ipsum dolor sit amet'
pubDate: 'May 17 2017'
tags: ["frontend", "css"]
---

本文的结构和理念参考自：http://web.jobbole.com/87164/

## 文章的初衷
现在的工作中往往依赖React.js，Backbone.js等等这样的framework来编写component，却对framework内部的逻辑知之甚少。于是在脱离开framework时，几乎难以仅仅使用vanilla JavaScript，HTML和CSS来单独进行开发。

这篇文章的目的是总结归纳，在不依赖framework时开发component的几个基本的步骤，使自己未来的工作更高效，并且更少地依赖于framework的逻辑，使得自己代码的复用率显著提高，相同地，也促进对framework原理和逻辑的理解。

## 为什么要开发component
原因非常简单：

1. 提高code maintainability
2. 提高code reusability

核心思想：DRY (don’t repeat your self)

## 具体实现
### requirement
假定我们要实现Facebook news feed card的设计：

![news feed cars](../../assets/write-component-1.png)


### breakdown
- Model：存储需要render这个news feed card的data（如发布者，标题，图片链接）以及储存一些前端使用的状态data（例如是否有点赞，是否播放当前视频）
- View：ui，可视部分
- Controller：逻辑处理，例如如何request评论data，如何send request

## 方法和步骤
### Model
Model是储存数据的地方，可以在这里做ajax call，也可以在controller内进行，取决于你的需求。

```typescript
const getNewsFeedModel = function() {
  return {
    title: 'news feed title',
    description: 'news feed description',
    imgSrc: 'http://placehold.it/350x150',  
  };
}
```

### View
View的核心是构建一个供component使用的template，这个template可以预先放置于HTML DOM tree中，也可以直接将其写于template function中。

#### 第一个方法：使用`<template>` tag
See https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template

```html
<template id="news-feed">
  <div class="news-feed">
    <div class="news-img">
      <img src="">
    </div>
    <article class="news-details">
      <h2></h2>
      <p></p>
    </article>
  </div>
</template>
<template>
```
 不同browser的支持状况不同，推荐使用前检查Can I use上的browser支持状况。

这里有一个小的变种，使用`<script type="text/template">` tag（Backbone在构建Backbone view的template时也使用这样的方法）。

因浏览器不能识别`text/template`这个MIME type，因此browser会忽略parse这个tag。借助这个特性，我们可以把template放入`<script>`tag内，供我们的javascript代码读取。代码示例如下：

```html
<script type="text/template" id="news-feed">
  <div class="news-feed">
    <div class="news-img">
      <img src="">
    </div>
    <article class="news-details">
      <h2></h2>
      <p></p>
    </article>
  </div>
</script>
```
在使用`<template>`tag时，我们只需要选择`#news-feed`，inject data，并生成和返回一份DOM Node Copy。

```typescript
const getNewsFeedView = function(model) {
  const template = document.querySelector('#news-feed');
  const title = template.content.querySelector('h2');
  const description = template.content.querySelector('p');
  const img = template.content.querySelector('img');
  title.text = model.title;
  description.text = model.description;
  img.src = model.imgSrc;
  // see https://developer.mozilla.org/en-US/docs/Web/API/Document/importNode
  return document.importNode(template.content, true);
}
```

#### 第二个方法：直接在view-generating function内写入template string

```typescript
const getNewsFeedView = function(model) {
  const template = [
    '<div class="news-feed">',
      '<div class="news-img">'
        `<img src="${model.imgSrc}">`
      '</div>',
      '<article class="news-details">',
        '<h2>',
          model.title,
        '</h2>'
        '<p>',
          model.description
        '</p>',
      '</article>',
    '</div>'
  ].join('');
  return
}
```
这种方法的缺点是代码的可维护性比较差，如果需要修改html markup，需要进行大量的修改，且不够直观。因此，更推荐使用第一种方法。

Controller
这部分的功能是在各个不同的event和state下，调用View的方法。例如click button to load more news feed。在这里可以attach各个DOM event。

## 尾声
经过本文的总结，我们可以发现，脱离开framework进行开发，其实很简单，也很强大。对我来说，最宝贵的一点是加深了对DOM的理解、以及如何高效和有效地剥离开各个部件，进行有效的代码利用。

阅读完本文后，不妨放下手中的framework，尝试用最简单的工具，从头构建一个component。相信你会受益匪浅。