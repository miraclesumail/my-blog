---
title: 'grid display advance'
description: 'grid进阶'
pubDate: 'Jan 08 2026'
heroImage: '../../assets/blog-placeholder-3.jpg'
---

<style>
    .small-code pre {
    font-size: 13px !important;  /* 强制覆盖默认大小 */
    line-height: 1.5 !important; /* 调整行高，让排版更紧凑 */
    padding: 1rem !important;    /* 缩小边距（可选） */
  }

  p {
    margin: 0 !important;
  }

  li {
     font-size: 16px !important;  /* 强制覆盖默认大小 */
    line-height: 1.4 !important; /* 调整行高，让排版更紧凑 */
    margin-bottom: 8px;
  }
</style>

1. 🤖 [水平垂直布局](#basic)
2. ⚙️ [跨行跨列排布](#tech)
3. 🔋 [autofill的用处](#autofill)
4. 🤸 [用grid快速排版布局](#layout)
5. 🕸️ [grid-auto-flow的使用](#snippets)


#### <a name="basic">🤖 水平垂直布局</a>

<div class="small-code">

```css
.gridWrap {
  display: grid;
  width: 500px;
  height: 500px;
  /** = grid-template-rows:repeat(2, 1fr) + grid-template-columns:repeat(3, 1fr) */
  grid-template: repeat(2, 1fr) / repeat(3, 1fr);
  /** 默认 */
  align-content: flex-start;
}

.gridWrap div {
  background: #1890ff;
}
```

```html
<div class="gridWrap">
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
</div>
```

</div>

#### <a name="tech">⚙️ 跨行跨列排布</a>

<div class="small-code">

```css
#grid {
  display: grid;
  height: 100px;
  /** 6行 总共有1-7个竖线 */
  grid-template-columns: repeat(6, 1fr);
  grid-template-rows: 100px;
}

/** 从1开始跨2列 */
#item1 {
  grid-column: 1 / span 2;
}

/** 从item2之后跨2行 */
#item2 {
  grid-column: span 2;
}

/** 从第7个线开始算1个 相当于 6/7 */
#item3 {
  grid-column: span 1 / 7;
}
```

</div>
展示如下图所示

![这是示例图片](/imgs/line.jpg)

#### <a name="autofill">🔋 autofill的用处</a>

<div class="small-code">

```css
.container {
  display: grid;
  width: 100%;
  height: 400px;
  /** 如果是auto-fit会把容器塞满 */
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  column-gap: 20px;
}
```

```html
<div class="gridWrap">
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
  <div></div>
</div>
```

</div>

1. 如果容器width = 6*150 + 5*20 = 1000, 那么刚好放下6个宽度为150
2. 当width = 7*150 + 6*20 = 1170放下7个div, 所以当width在[1000, 1170]的区间每个div宽度为1fr

![这是示例图片](/imgs/auto-fill.png)

#### <a name="layout">⚙️ 用grid快速排版布局</a>

<div class="small-code">

```css
.layout {
  display: grid;
  width: 100%;
  height: 600px;
  grid-template-columns: 200px 1fr 150px;
  grid-template-rows: 120px 1fr 150px;
  /** .不显示 */
  grid-template-areas:
    'header header .'
    'nav main side'
    'footer footer footer';
}

header {
  grid-area: header;
}
nav {
  grid-area: nav;
}
main {
  grid-area: main;
}
aside {
  grid-area: side;
}
footer {
  grid-area: footer;
}
```

```html
<div class="layout">
  <header>头部</header>
  <nav>导航</nav>
  <main>主页</main>
  <aside>侧边栏</aside>
  <footer>页脚</footer>
</div>
```

</div>

![这是示例图片](/imgs/layout.jpg)

#### <a name="snippets">⚙️ grid-auto-flow的使用</a>

<div class="small-code">

```css
#grid {
  height: 200px;
  width: 200px;
  display: grid;
  gap: 10px;
  grid-template: repeat(3, 1fr) / repeat(3, 1fr);
  /** row是默认值 */
  grid-auto-flow: row;
}

#item1 {
  background-color: lime;
  grid-column: span 2;
}

#item2 {
  background-color: yellow;
  grid-column: span 2;
}

#item3 {
  background-color: blue;
}

#item4 {
  background-color: red;
}

#item5 {
  background-color: aqua;
}
```

```html
<div id="grid">
  <div id="item1"></div>
  <div id="item2"></div>
  <div id="item3"></div>
  <div id="item4"></div>
  <div id="item5"></div>
</div>
```

</div>

1. grid-auto-flow: row; 横向排列放不下的起新的一行
![这是示例图片](/imgs/eg1.jpg)

2. grid-auto-flow: row dense; 会自动填满横排
![这是示例图片](/imgs/eg3.jpg)

3. grid-auto-flow: column; 以竖直向排列
![这是示例图片](/imgs/eg2.jpg)




