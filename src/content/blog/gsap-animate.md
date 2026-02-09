---
title: 'gsap animate template'
description: 'flex进阶'
pubDate: 'Jan 06 2026'
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

#### 📋 <a name="table">Table of Contents</a>

1. 🤖 [基础展示](#basic)
2. ⚙️ [进阶用法](#tech)
3. 🔋 [stagger的使用](#features)
4. 🤸 [scrollTrigger插件的使用](#scroll-trigger)
5. 🕸️ [序列动画timeline](#snippets)
6. 🔗 [如何手动控制animation](#links)
7. 🚀 [gsap通用utility函数](#more)
8. 🚀 [svg动画的应用](#svg)

#### <a name="basic">🤖 基础展示</a>

<div class="small-code">

```js
import gsap from 'gsap';
// 将circle从(0,0)移动到(40, 0) 背景色变为blue
gsap.to('.circle', { x: 40, fill: 'blue' });

// 将circle从(-40,0)移动到(0, 0) 背景色从blue变为0
gsap.from('.circle', { x: -40, fill: 'blue' });

// 从(-40, 0)到(40, 0) blue并未green
gsap.fromTo('.circle', { x: -40, fill: 'blue' }, { x: 40, fill: 'green' });

// 直接设置属性没有动画
gsap.set('.circle', { x: 40, fill: 'blue' });

// 可以同时设置多个target
let square = document.querySelector('.square');
let circle = document.querySelector('.circle');

// -1无限重复 ease='none'是linear
gsap.to([square, circle], { x: 200, duration: 0.2, repeat: -1, ease: 'none' });
```

</div>

#### <a name="tech">⚙️ 进阶用法</a>

<div class="small-code">

```js
import gsap from 'gsap';

// x轴距离相对+100  yoyo=true来回重复动画
gsap.to('.circle', { x: '+=100', duration: 1, repeat: 3, yoyo: true });

// 可以同样对obj使用
let obj = { myNum: 10, myColor: 'red' };

gsap.to(obj, {
  myNum: 200,
  myColor: 'blue',
  // 每一帧触发
  onUpdate: () => console.log(obj.myNum, obj.myColor),
});

let position = { x: 0, y: 0 };

function draw() {
  // erase the canvas
  ctx.clearRect(0, 0, 300, 300);
  // redraw the square at it's new position
  ctx.fillRect(position.x, position.y, 100, 100);
}

//animate x and y of point
gsap.to(position, {
  x: 200,
  y: 200,
  duration: 4,
  // unlike DOM elements, canvas needs to be redrawn and cleared on every tick
  onUpdate: draw,
});
```

</div>

#### <a name="features">⚙️ stagger的使用</a>

操作同一类元素进行有序动画

<div class="small-code">

```html
<div class="section">
  <div class="box green"></div>
  <div class="box purple"></div>
  <div class="box orange"></div>
  <div class="box purple"></div>
  <div class="box green"></div>
</div>
```

```js
gsap.to('.box', {
  duration: 1,
  rotation: 360,
  opacity: 1,
  delay: 0.5,
  stagger: 0.1, // stagger in from the left with a 0.1 second gap in between animations
  ease: 'sine.out',
});

document.querySelectorAll('.box').forEach((box, index) => {
  box.addEventListener('click', () => {
    gsap.to('.box', {
      duration: 0.5,
      opacity: 0,
      y: -100,
      stagger: {
        from: index, // 按下的元素的inde
        amount: 1, // 向2边扩散
        ease: 'back.in',
        overwrite: 'auto',
      },
    });
  });
});

// 可以自定义时间 偶数时是index*0.1
gsap.to('.box', {
  y: 100,
  stagger: function (index, target, list) {
    // your custom logic here. Return the delay from the start (not between each)
    return index % 2 ? index * 0.1 : index + 0.1;
  },
});
```

</div>

#### <a name="scroll-trigger">🤸 scrollTrigger插件的使用</a>

使用滚动插件实现特定时机动画

<div class="small-code">

```js
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);

// 当窗口滚动100px触发
gsap.to('.box', {
  x: 100,
  scrollTrigger: {
    trigger: 'box',
    start: 100,
  },
});

// 当box的上边缘滚动到距离窗口上面100px触发
gsap.to('.box', {
  x: 100,
  scrollTrigger: {
    trigger: '.box',
    start: 'top 100px',
    // start: 'bottom center',  底部距离视口上面50vh触发
    markers: true,
  },
});
```

使用toggleActions进行出入时机事件设置

```js
/**
 * start： number | string 默认是box的top距离viewport顶部距离(number) 可以自定义'top center'当box的中部距离viewpoint=0.5vh时
 * end： number | string 默认是box的bottom距离viewport顶部距离(number) 可以自定义'bottom center'当box的中部距离viewpoint=0.5vh时
 */
gsap.to('.box', {
  x: 600,
  duration: 10,
  scrollTrigger: {
    trigger: '.box',
    start: 'top center',
    end: () => `+=${box.offsetHeight}`, // 用函数当窗口大小改变时可以refresh
    toggleActions: 'play pause',
    markers: true,
  },
});

/**
 * toggleActions: 'play pause resume reset'
 * 1. 当start触发时play 对应onenter
 * 2. 当end触发时pause  onleave
 * 3. 当再次回到end时resume动画  onEnterBack
 * 4. 当再次回到start时reset到初始位置  onLeaveBack
 */
gsap.to('.box', {
  x: 600,
  duration: 10,
  scrollTrigger: {
    trigger: '.box',
    start: 'top center',
    end: 'bottom center',
    toggleActions: 'play pause resume reset',
    onLeaveBack: (self) => {
      console.log('onLeaveBack');
    },
    onEnterBack: (self) => {
      console.log('onEnterBack');
    },
    // once: true,
    // onEnter: () => void,  当到了start的时候
    //  onLeave: () => void  当到了end的时候
    markers: true,
  },
});

// 创建一个初始为paused的animation
const anim = gsap.to(box, { x: 800, duration: 20, paused: true, ease: 'none' });

ScrollTrigger.create({
  trigger: box,
  start: 'center 70%',
  onEnter: () => anim.play(),
});

ScrollTrigger.create({
  trigger: box,
  start: 'top bottom',
  end: 'bottom bottom',
  onEnterBack: () => anim.pause(),
});
```

</div>

#### <a name="snippets">🕸️ scrollTrigger插件的使用</a>

使用timeline创建时间序列动画

<div class="small-code">

```js
// 这个动画初始是暂停的
const tl = gsap.timeline({
  repeat: -1,
  paused: false,
  onUpdate,
  reversed: true,
});

console.log(tl.duration()); // 4

const onUpdate = () => {
  // 进度 0.8
  console.log(tl.progress);
  // progress * duration
  console.log(tl.time());
};

tl.to('#green', { x: width, xPercent: -100, duration: 2 })
  .to('#purple', { x: width, xPercent: -100, duration: 1 })
  // 第2，3组动画同时进行
  .to('#orange', { x: width, xPercent: -100, duration: 1 }, '<');
// 第三组开始时间 = 2组duration - 0.9  就是第二组开始0.1s后第三组开始
  .to('#orange', { x: width, xPercent: -100, duration: 1 }, '-=0.9');

// 需要手动播放
tl.play();
```

</div>

#### <a name="links">🔗 如何手动控制animation</a>

<div class="small-code">

```js
let nav = document.querySelector('.nav');

const tween = gsap.to('.flair', {
  duration: 2,
  x: () => nav.offsetWidth,
  xPercent: 0, // offset by the width of the box
  rotation: 90,
  ease: 'none',
  paused: true,
});

//pause
tween.pause();

//resume (honors direction - reversed or not)
tween.resume();

//reverse (always goes back towards the beginning)
tween.reverse();

// 跳到0.5秒处
tween.seek(0.5);

// 跳到进度为25%位置
tween.progress(0.25);

// 调整为半速执行
tween.timeScale(0.5);

//make the tween go double-speed
tween.timeScale(2);

//immediately kill the tween and make it eligible for garbage collection
tween.kill();

// You can even chain control methods
// Play the timeline at double speed - in reverse.
tween.timeScale(2).reverse();

// onComplete: invoked when the animation has completed.
// onStart: invoked when the animation begins
// onUpdate: invoked every time the animation updates (on every frame while the animation is active).
// onRepeat: invoked each time the animation repeats.
// onReverseComplete: invoked when the animation has reached its beginning again when reversed.
```

</div>

#### <a name="more">🚀 gsap通用utility函数</a>

常见的开发utility工具函数

<div class="small-code">

clamp的使用

```js
// 100 > 70 最大显示70
const val = gsap.utils.clamp(30, 100, 70); // 70

gsap.set('#ball', { xPercent: 0, x: 50 });

const num = document.querySelector('#num');

window.addEventListener('keyup', (e) => {
  gsap.set('#ball', {
    x: gsap.utils.clamp(30, 70, num.value),
  });
});
```

```js
let distributor = gsap.utils.distribute({
  base: 5, // 基础值（起点值）
  amount: 10, // 总增量（最大变化范围）
  from: 'center', // 从哪里开始计算距离（"center" 表示中间是起点）
  ease: 'power1.inOut', // 分布的曲线（非线性分布）
});

// 假设有5个.box  中间第一个为5， 2边的是15(5+10)
gsap.to('.box', {
  scale: distributor,
});
```

- **数值范围**：结果将在 50 到 150 之间变化（Base 50 + Amount 100）
- **分布逻辑**：因为 from: "center"，所以：
  - 离中心最近的元素（Center），得到的值最接近 base (50)。
  - 离中心最远的元素（Edges，即数组的头和尾），得到的值最接近 base + amount (150)。

```js
//interpolate halfway between 0 and 500 (number)
gsap.utils.interpolate(0, 500, 0.5); // 250

// strings
gsap.utils.interpolate('20px', '40px', 0.5); // "30px"

//colors
gsap.utils.interpolate('red', 'blue', 0.5); // "rgba(128,0,128,1)"

//objects
gsap.utils.interpolate({ a: 0, b: 10, c: 'red' }, { a: 100, b: 20, c: 'blue' }, 0.5); // {a: 50, b: 15, c: "rgba(128,0,128,1)"}
```

```js
var output = func1(func2(func3(input)));

// cleaner with pipe()
var transfrom = gsap.utils.pipe(func1, func2, func3);
var output = transform(input);

var transformer = gsap.utils.pipe(
  // clamp between 0 and 100
  gsap.utils.clamp(0, 100),

  // then map to the corresponding position on the width of the screen
  gsap.utils.mapRange(0, 100, 0, window.innerWidth),

  // then snap to the closest increment of 20
  gsap.utils.snap(20),
);

// now we feed one value in and it gets run through ALL those transformations!:
transformer(25.874);
```

</div>

#### <a name="svg">🚀 svg动画的应用</a>

<div class="small-code">

```js
import { MotionPathPlugin } from 'gsap/MotionPathPlugin';
import { DrawSVGPlugin } from 'gsap/DrawSVGPlugin';

gsap.registerPlugin(MotionPathPlugin);
gsap.registerPlugin(DrawSVGPlugin);

// #div划出一道path的轨迹  align是指沿着谁画轨迹
gsap.to('#div', {
  duration: 5,
  repeat: 12,
  repeatDelay: 3,
  yoyo: true,
  ease: 'power1.inOut',
  motionPath: {
    path: '#path',
    align: '#path',
    autoRotate: false, // 是否跟着path的弧度改变朝向
    alignOrigin: [0.5, 0.5], // div的正中心一直都沿着轨迹的弧线
  },
});

// 从原点画到(totalLength - 100)
gsap.to('#path', { drawSVG: 100, duration: 5, ease: 'power1.inOut' });

// 从原点画到(totalLength - 0)画满一圈
gsap.to('#path', { drawSVG: 0, duration: 5, ease: 'power1.inOut' });

// 从原点画到(totalLength - 20%) 就是从0-80%
gsap.to('#path', { drawSVG: '20%', duration: 5, ease: 'power1.inOut' });

// 从原点画到(totalLength - 100%) 就是从0-0根本不画
gsap.to('#path', { drawSVG: '100%', duration: 5, ease: 'power1.inOut' });

// 假设pathLength = 200, 从结束点0开始算  30% - 70%不画
gsap.to('#path', { drawSVG: '30% 70%', duration: 5, ease: 'power1.inOut' });

// 假设pathLength = 200, 从结束点0开始算  50-120不画 [200 - 150]画  [0-80]画
gsap.to('#path', { drawSVG: '50 120', duration: 5, ease: 'power1.inOut' });

// 使用keyframes使得镜头来回无间隙切换
gsap.to(getCamera().position, {
    keyframes: {
      '0%': { z: 80 },
      '25%': { z: 100 },
      '50%': { z: 80 },
      '75%': { z: 60 },
      '100%': { z: 80 },
    },
    duration: 1,
    repeat: -1,
    ease: 'none',
  });
```

</div>
