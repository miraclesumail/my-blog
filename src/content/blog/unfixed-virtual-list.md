---
title: 'how to implement unfixed-virtual-list'
description: 'Lorem ipsum dolor sit amet'
pubDate: 'Jan 01 2026'
heroImage: '../../assets/blog-placeholder-4.jpg'
---

<style>
    .small-code pre {
    font-size: 13px !important;  /* 强制覆盖默认大小 */
    line-height: 1.5 !important; /* 调整行高，让排版更紧凑 */
    padding: 1rem !important;    /* 缩小边距（可选） */
  }
</style>

#### 如何实现不固定高度的虚拟列表

##### 1.具体思路如下

1.  **计算当前开始索引startIndex**：无论是固定高度还是非固定, 都需要根绝滚动距离计算出starIndex
2.  **计算渲染结束索引endIndex**：粗略算出应该为startIndex + Math.floor(容器高度 / item预估高度) + bufferCount(缓冲count预防快速滚动出现白屏)
3.  **初始化listData每个item的位置信息**：定义一个positionsRef, 按照item预估高度estimatedHeight计算出每个item的top、bottom信息
4.  **渲染当前可见visibleData纠正位置信息**：上面定义的positionsRef记录的是相同高度算出来的位置信息，实际需要将visibleData通过map渲染到实际的div上, 通过得到每个item的真实高度去修改positionsRef记录的位置信息
5.  **计算滚动容器的总高度和每个item的位移**：容器的总高度是个动态值, 每次滚动触发更新startIndex的值时，都有可能改变总高度，总高度实际就等于positionsRef最后一个元素的bottom值, 然后visibleData中每个item直接按照计算好的top设置translateY=top即可

##### 2.初始化positionsRef

<div class="small-code">

```js
// 展示列表的数据源
const listData = [{}];
// 假设预估高度是50
const ESTIMATED_HEIGHT = 50;
const positionsRef = useRef(
  listData.map((_, index) => ({
    index,
    height: ESTIMATED_HEIGHT,
    top: index * ESTIMATED_HEIGHT,
    bottom: (index + 1) * ESTIMATED_HEIGHT,
  })),
);
```
</div>


positionsRef记录了初始化所有item的位置信息

##### 3.根据scrollTop计算出startIndex和endIndex

<div class="small-code">

```js
// 假设容器高度为600
const containerHeight = 600;
// 添加bufferCount预留预防快速滚动白屏
const BUFFER_COUNT = 5;
// 当前滚动高度
const [scrollTop, setScrollTop] = useState(0);

const onScroll: UIEventHandler<HTMLDivElement> = (e) => {
    // 使用 requestAnimationFrame 节流
    requestAnimationFrame(() => {
      setScrollTop((e.target as HTMLDivElement).scrollTop);
    });
  };

function findStartIndex(positions, scrollTop) {
  // ...此处省略，可以用二分查找获取startIndex
  return startIndex;
}

// 根据上面算出的positionsRef和scrollTop得到startIndex
const startIndex = findStartIndex(positionsRef, scrollTop);
const endIndex = Math.min(
    startIndex! + Math.ceil(600 / ESTIMATED_HEIGHT), // 假设视口高度600
    listData.length - 1
  );

// 加上缓冲区
  const renderStartIndex = Math.max(0, startIndex! - BUFFER_COUNT);
  const renderEndIndex = Math.min(listData.length - 1, endIndex + BUFFER_COUNT);
```
</div>


实际渲染的区间就是[renderStartIndex, renderEndIndex]

##### 4.渲染visibleData获取itemRefs

<div class="small-code">

```js
// 测量 DOM 节点的引用 这个很重要用于更正positionsRef
const itemRefs = useRef < any > {};
// 获取当前要渲染的数据片段;
const visibleData = listData.slice(renderStartIndex, renderEndIndex).map((item, index) => ({
  ...item,
  _index: renderStartIndex + index, // 保存原始索引
}));

// 将visibleData渲染出来，把每个el设置到itemRefs
const Container = () => {
  return (
    <>
      {visibleData.map((item) => {
        const index = item._index;
        const currentPos = positionsRef.current[index];

        return (
          <div
            key={item.id}
            // @指向真实的div
            ref={(el) => (itemRefs.current[index] = el)}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              // 使用 transform 性能更好
              transform: `translateY(${currentPos.top}px)`,
            }}
          >
            {renderItem(item)}
          </div>
        );
      })}
    </>
  );
};
```
</div>


上面获取到了visibleData渲染后每个真实item，接下来用itemRefs去更新positionsRef

##### 5.通过itemRefs获取每个item元素计算更新位置信息

<div class="small-code">

```js
//  用于强制更新组件（当高度修正后）
const [, forceUpdate] = useState({});

// visibleData变动时更新
useEffect(() => {
  if (checkChanged()) {
    updatePositionsRef();
  }
}, [visibleData]);

// 检查实际渲染的item的高度是否和预设不一样
function checkChanged() {
  const nodes = itemRefs.current;
  // 标记是否需要更新
  let hasChanged = false;

  Object.keys(nodes).forEach((key) => {
    const node = itemRefs.current[key];
    // 真实的item节点高度
    const height = rect.height;
    // 预设高度
    const oldHeight = positionsRef.current[index].height;

    // 如果真实高度与缓存高度不一致，需要修正
    if (Math.abs(height - oldHeight) > 0.5) {
      // 0.5容错
      positionsRef.current[index].height = height;
      positionsRef.current[index].bottom = positionsRef.current[index].top + height;
      hasChanged = true;
    }
  });
  return hasChanged;
}

function updatePositionsRef() {
  // 无论怎么变，第一个元素的top都不会变是0
  let currentTop = positionsRef.current[0].top;

  for (let i = 0; i < positionsRef.current.length; i++) {
    const item = positionsRef.current[i];
    item.top = currentTop;
    item.bottom = currentTop + item.height;
    currentTop = item.bottom;
  }

  // 强制更新dom， ref的值改变不会导致render
  forceUpdate({});
}
```
</div>


这一步最关键，决定了不固定item是否正常渲染

##### 6.修改jsx返回的render内容

<div class="small-code">

```js
// 计算总高度（用于撑开滚动条）
const totalHeight = positionsRef.current[positionsRef.current.length - 1].bottom;

// Content是虚拟列表组件渲染的全部内容
const Content = () => {
  const style = { height: totalHeight, position: 'relative' };
  return (
    <div
      onScroll={onScroll} // 对应上面的滚动监听
      style={{
        height: '600px', // 视口高度, 看情况确定
        overflowY: 'auto',
        position: 'relative',
        border: '1px solid #ccc',
      }}
    >
      <div style>
        {/* 第四步中的Container */}
        <Container />
      </div>
    </div>
  );
};
```
</div>


##### 至此已实现全部逻辑

- <a href="https://www.npmjs.com/package/@miracle_sumail/react-virtual-list" target="_blank" rel="noopener noreferrer">点击下载查看完整code</a>
