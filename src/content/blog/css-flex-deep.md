---
title: 'css flex deep explain'
description: 'flex进阶'
pubDate: 'Jul 08 2025'
heroImage: '../../assets/blog-placeholder-3.jpg'
---

#### 使用flex计算div实际宽度

##### 1.当子容器宽度加起来小于父容器宽度时
<style>
    .small-code pre {
    font-size: 13px !important;  /* 强制覆盖默认大小 */
    line-height: 1.4 !important; /* 调整行高，让排版更紧凑 */
    padding: 1rem !important;    /* 缩小边距（可选） */
  }
</style>

<style>
    .js-code pre {
    font-size: 15px !important;  /* 强制覆盖默认大小 */
    line-height: 1.5 !important; /* 调整行高，让排版更紧凑 */
    padding: .5rem !important;    /* 缩小边距（可选） */
  }
</style>

<div class="small-code">

```html
<!DOCTYPE html>
<html>
  <style>
    .container {
      width: 600px;
      display: flex;
    }

    .left {
      flex: 2 1 300px;
    }

    .right {
      flex: 1 2 200px;
    }
  </style>
  <body>
    <div class="container">
      <div class="left"></div>
      <div class="right"></div>
    </div>
  </body>
</html>
```

</div>

##### 上述代码div.left和div.right在页面中宽度分别是多少？

<div class="small-code">

```html
   flex: flex-grow flex-shrink flex-basis; (分配比例, 伸缩比例, 基础宽度)
```
</div>

##### flex的属性描述如上，flex-grow,flex-basis在接下来计算中有作用

</br>

##### 下面展示计算方法，当子div宽度之和小于父容器宽度(200px + 300px < 600px)的时候
<div class="js-code">

```javascript
   const parentWidth = 600;
   const childrenWidth = divLeft(300) + divRight(200) = 500;
   
   每个div的实际宽度为 = flex-basis + (parentWidth - childrenWidth)*(flex-grow/(flex-grow1 + ... + flex-growN));

   带入上述公式计算
   const divLeft = 300 + (600 - 500)*(2/3)
   const divRight = 200 + (600 - 500)*(1/3)
```
</div>

##### 最后计算出来divLeft = 366.67  divRight = 233.33, 注意flex-basis为自己flex属性上最后一个值, divLeft是300，divRight是200 
</br>

##### 2.当子容器宽度加起来大于父容器宽度时
<div class="small-code">

```html
<!DOCTYPE html>
<html>
  <style>
    .container {
      width: 600px;
      display: flex;
    }

    .left {
      flex: 2 1 300px;
    }

    .right {
      flex: 1 2 400px;
    }
  </style>
  <body>
    <div class="container">
      <div class="left"></div>
      <div class="right"></div>
    </div>
  </body>
</html>
```

</div>

##### 上述代码div.left和div.right在页面中宽度分别是多少？

<div class="small-code">

```html
   flex: flex-grow flex-shrink flex-basis; (分配比例, 伸缩比例, 基础宽度)
```
</div>

##### flex的属性描述如上，因为子div宽度和超过了父容器，flex-shrink,flex-basis在接下来计算中有作用

</br>

##### 下面展示计算方法，当子div宽度之和大于父容器宽度(300px + 400px > 600px)的时候
<div class="js-code">

```javascript
   const parentWidth = 600;
   const childrenWidth = divLeft(300) + divRight(400) = 700;
   const shrinkRatio = (flex-shrink * flex-basis)/(flex-shrink1 * flex-basis1 + ... + flex-shrinkN * flex-basisN)
   
   每个div的实际宽度为 = flex-basis - (childrenWidth - parentWidth)*shrinkRatio;

   带入上述公式计算
   const divLeft = 300 - 100 * (1*300 / (1*300 + 2*400))
   const divRight = 400 - 100 * (2*400 / (1*300 + 2*400))
```
</div>

##### 最后计算出来divLeft = 272.72  divRight = 327.27 , 注意flex-shrink为自己flex属性上第二个值, divLeft是1，divRight是2

</br>

#### 总结
1. 当所有子div宽度和小于父容器宽度，根据（父容器宽度-子div宽度和）结合每个子div的flex-basis和flex-grow去计算
2. 当所有子div宽度和大于父容器宽度，根据（子div宽度和-父容器宽度）结合每个子div的flex-basis和flex-shrink去计算

</br>

#### 思考: 如果flex-basis写成百分比30%或者0，宽度又该如何计算？

  

