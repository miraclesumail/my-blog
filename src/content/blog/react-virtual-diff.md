---
title: 'react-virtual-diff'
description: 'diff算法'
pubDate: 'Jan 10 2026'
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

  .bold {
    font-weight: bolder;
  }

  li {
     font-size: 16px !important;  /* 强制覆盖默认大小 */
    line-height: 1.4 !important; /* 调整行高，让排版更紧凑 */
    margin-bottom: 8px;
  }

   h4{
    margin-top: 20px;
   }

  th, td {
    border: 1px solid #ccc;
     font-size: 16px;
  }
</style>

1. 🤖 [diff比较过程](#diff)
2. ⚙️ [lastPlaceIndex判断移动](#tech)
3. 🔋 [算法的分析](#analyze)
4. 🤸 [fiber解读](#fiber)
5. 🕸️ [对比vue2的双端比较方法](#vue2)
6. 🔗 [react和vue2的diff对比](#links)
6. 🚀 [最长递增子序列](#seq)

#### <a name="diff" class="title">🤖 diff比较过程</a>

##### 1： 第一轮遍历：寻找"原地复用"的节点

1. 同时遍历新旧两个列表
2. 判断key和type类型，如果都一样可以复用，继续比对下一个
3. 遇到不匹配(key不一样)第一轮遍历立即停止

##### 2： 第二轮遍历：处理移动的节点

1. 建立map映射: react会把旧列表剩余的没比对完的fiber节点塞进一个以key为键的map里面
2. 继续遍历新列表中剩下的节点，拿着新节点的key去刚才那个map映射表里搜寻，如果找到了则复用，并从map中删除
3. 新列表遍历完后，如果map里面还有剩下的旧节点，说明它们已经不在了，直接卸载

#### <a name="tech" class="title">⚙️ lastPlaceIndex判断移动</a>

假设有组节点ABCDEFGH，新节点为ACBDGEFH(复杂度为o(n)一轮)
| 步骤 | 处理节点 | 旧索引 | lastplaceIndex | 动作 | 说明 |
| :---: | :----:| :---: | :---: | :---: | :---: |
| 1 | A | 0 | 0 | 不动 | 0>= 0 更新lastplaceIndex=0 |
| 2 | C | 2 | 0 | 不动 | 2>= 0 更新lastplaceIndex=2 |
| 3 | B | 1 | 2 | 移动 | 1 < 2 B在C前面，现在跑到后面去了 |
| 4 | D | 3 | 2 | 不动 | 3 > 2 更新lastplaceIndex=3 |
| 5 | G | 6 | 3 | 不动 | 6 > 2 更新lastplaceIndex=6 |
| 6 | E | 4 | 6 | 移动 | 4 < 6 E在G前面，现在跑到后面去了 |
| 7 | F | 5 | 6 | 移动 | 5 < 6 F在G前面，现在跑到后面去了
| 8 | H | 7 | 6 | 不动 | 7 > 6 更新lastplaceIndex=7 |

1. 第一个需要移动的是B, 寻找在新节点中位于它右边第一个不需要移动的是D
2. 第一个需要移动的是E, 寻找在新节点中位于它右边第一个不需要移动的是H
3. 第一个需要移动的是F, 寻找在新节点中位于它右边第一个不需要移动的是H
4. 所以一共需要进行3次移动, 第一次parent.insertBefore(B, D), 第二次parent.insertBefore(E, H), 第三次parent.insertBefore(F，H)
5. 如果需要移动节点的右边没有节点了，那就是parent.append(需要移动的node)

#### <a name="analyze" class="title">🔋 算法的分析</a>

fiber的节点如下:

<div class="small-code">

```js
    {
        type: 'div',
        key: 'A',
        child: FiberNodeB,  // 第一个子节点
        sibling: FiberNodeC,  // 下一个兄弟节点
        child: FiberNodeP,   // 父节点
    }
```

</div>
因为每个节点只知道它的右边是谁（sibling），不知道左边是谁，所以它：

1. 不能像 Vue 那样做双端对比：双端对比需要同时从两头往中间扫，但链表没法从右往左跳。
2. 必须依赖 lastPlacedIndex：既然只能从左往右看，那就记下一个“水位线”，只要新节点在旧家谱里的位置比水位线低，就说明它“插队”了，需要挪窝

需要注意的是react在diff完后先把所有需要移动的节点标记好，最后再一次性更新ui

#### <a name="fiber" class="title">🤸 fiber解读</a>

fiber解决的问题

- 痛点：主线程被霸占
  - 旧版采用递归的方式对比虚拟dom。如果组件很深，js引擎会持续占据主线程长达100ms甚至更多

- 后果：视觉上的卡死
  - 浏览器是一个单线程，每秒需要刷新60次来处理动画和输入。如果js逻辑超过16ms，浏览器没时间响应用户点击和绘制动画

关键代码:

<div class="small-code">

```js
function workLoopConcurrent() {
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}

function shouldYield() {
  const elapsedTime = getCurrentTime() - startTime;

  if (elapsedTime < 5) {
    return false;
  }
  return true;
}
```

</div>

- 处理fiberA(耗时0.1ms) --> 检查shouldYield()(还没到5ms, 继续)
- 处理fiberB(耗时0.2ms) --> 检查shouldYield()(还没到5ms, 继续)
- 处理fiberC(耗时3ms) --> 检查shouldYield()(耗时3ms, 总共3.3ms，还没到5ms, 继续)
- 处理fiberD(耗时2ms) --> 检查shouldYield()(共5.3ms, 超过5ms)
- 立即停下，将控制权交给浏览器

1. 如果有更高优先级的任务(点击、滚动、动画): 会放弃刚才进行fiberE的任务进入处理高优先的
2. 如果没有新任务：react会在下一次浏览器空闲时继续处理刚才那个被中断的任务(fiber记录之前的指针)

#### <a name="vue2" class="title">🕸️ 对比vue2的双端比较方法</a>

##### 1.核心机制：四种比较路径

每一轮循环会按照以下优先级进行四次比较：

1. 旧头 vs 新头
2. 旧尾 vs 新尾
3. 旧头 vs 新尾
4. 旧尾 vs 新头

<p class="bold">实例演示: ABCDEFG VS ACBDGEF</p>

| 轮次 |         比较动作         |     结果     |  移动  |                   说明                    |
| :--: | :----------------------: | :----------: | :----: | :---------------------------------------: |
|  1   |           A-A            |     匹配     |  不动  |           oS移动到B, nS移动到C            |
|  2   |     B-C,G-F,B-F,G-C      |   都不匹配   |  移动  | 把C移动到oS(B)前面，此时为A,C,[B,D,E,F,G] |
|  3   |      oS(B) - nS(B)       |     匹配     |  不动  |        oS移动到D, A,C,B,[D,E,F,G]         |
|  4   |      oS(D) - nS(D)       |     匹配     |  不动  |           oS移动到E, nS移动到G            |
|  5   |     E-G,G-F,E-F,G-G      | 旧尾新头匹配 |  移动  |     oE(G)移动到E前面, A,C,B,D,[G,E,F]     |
|  6   | oS(E)-nS(E), oE(F)-nE(F) |    全匹配    | 不移动 |                 对比完成                  |

<p class="bold">双端比较总共移动2次，如果是react diff需要移动3次</p>

#### <a name="links" class="title">🔗 react和vue2的diff对比</a>

1. 在上述ABCDEFG VS ACBDGEF的对比中vue2移动次数少
2. 如果是ABCDEFGH VS CFGEBHAD对比，vue2移动次数会比react多

<p class="bold">这说明了什么?</p>

- <span class="bold">双端diff：</span>在处理局部微调时具有比较好的效果
- <span class="bold">单向diff：</span>在处理整体乱序时，效果会比较好

vue2的双端比较通过牺牲多一点js计算量换取更少的dom改变，而在web性能中dom变动的代价通常大于js的逻辑

#### <a name="seq" class="title">🚀 最长递增子序列</a>
以ABCDEFG和BACDEGF为例(这个算法复杂度为nlogn)

<p class="bold">第一阶段：预处理</p>

- <span class="bold">旧列表：</span> [A,B,C,D,E,F,G]
- <span class="bold">新列表：</span> [B,A,C,D,E,G,F]

1. 旧A VS 新B
2. 旧G VS 新F

预处理没能排除首尾节点

<p class="bold">第二阶段：构建映射表</p>

1. 新节点[B,A,C,D,E,G,F]在老的列表对应索引是[1,0,2,3,4,6,5]
2. 取[0,2,3,4,6]为最长递增序列
3. 从新节点末尾开始计算，F的旧索引5不在递增序列里，需要移动F
4. G,E,D,C,A都在序列当中不需要移动
5. B原始索引1不在序列中，需要移动

<p class="bold">F在新节点中后面没有元素，因此是container.append(F), B在新节点里是在A前面，是container.insertBefore(B,A)</p>








