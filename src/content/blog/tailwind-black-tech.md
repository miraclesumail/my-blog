---
title: 'tailwind-black-tech'
description: 'Lorem ipsum dolor sit amet'
pubDate: 'Jan 03 2026'
heroImage: '../../assets/blog-placeholder-4.jpg'
---

### 📋 <a name="table">Table of Contents</a>

1. 🤖 [自定义颜色和距离](#introduction)
2. ⚙️ [添加自定义类名](#tech-stack)
3. 🔋 [悬浮聚焦状态改变子元素](#features)
4. 🤸 [兄弟元素状态改变如何影响](#quick-start)
5. 🕸️ [子元素根据父容器宽度展示](#snippets)
6. 🔗 [根据js变量动态展示类名](#links)
7. 🚀 [使用utility自定义类名](#more)
8. 🚀 [使用变体动态编写样式](#variant)

#### <a name="introduction">🤖 自定义颜色和距离</a>

```css
@theme {
  /* --- 1. 自定义颜色 --- */

  /* 定义 --color-primary */
  /* 使用时对应类名：bg-primary, text-primary, border-primary */
  --color-primary: #3b82f6;

  /* 你也可以使用现代的 oklch 颜色空间（v4 推荐） */
  --color-brand: oklch(0.6 0.15 200);

  /* 定义不同深度的颜色 (bg-primary-500) */
  --color-primary-100: #dbeafe;
  --color-primary-500: #3b82f6;
  --color-primary-900: #1e3a8a;

  /* --- 2. 自定义间距 (Space / Margin / Padding) --- */
  --spacing-1: 8px;

  /* 
     注意：覆盖了 1 之后，原来的 2 (默认也是 8px) 就和 1 一样大了。
     如果你希望整个系统都翻倍 (1=8px, 2=16px)，你需要手动覆盖常用的几个值：
  */
  --spacing-2: 16px;
  --spacing-3: 24px;
  --spacing-4: 32px;
  --spacing-4_5: 36px;
}
```

注意在 Tailwind v4 中，没有一个所谓的“基础系数变量”（比如 base-unit）能让你改一个数字就自动更新所有间距。所有的间距（1, 2, 3...）都是独立的 CSS 变量。

```html
<!-- 使用 bg-primary -->
<button class="bg-primary text-white px-4 py-2 rounded">主要按钮</button>

<!-- 使用 bg-brand -->
<div class="bg-brand text-primary-900">品牌色区块</div>

<!-- 使用 space-x-4.5 (对应你定义的 36px) -->
<div class="flex space-x-4_5">
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</div>
```

#### <a name="tech-stack">🤖 添加自定义类名</a>

```css
@import 'tailwindcss';

/* 放在 @import 之后 */
@layer components {
  .btn-primary-custom {
    /* 
      1. 使用 @apply 组合 Tailwind 工具类 
      2. 也可以混合写原生 CSS
    */
    @apply pt-2 mx-2 bg-blue-500 text-white rounded;

    /* 混合原生 CSS 是合法的，且推荐用于复杂的交互 */
    transition: transform 0.2s;
  }

  /* 添加修饰符状态 */
  .btn-primary-custom:hover {
    @apply bg-blue-600;
    transform: scale(1.05);
  }
}
```

直接在div中使用类名btn-primary-custom

#### <a name="features">🤖 悬浮聚焦状态改变子元素</a>

```html
<ul role="list">
  {#each people as person}
  <li class="group/item ...">
    <!-- ... -->
    <a class="group/edit invisible group-hover/item:visible ..." href="tel:{person.phone}">
      <span class="group-hover/edit:text-gray-700 ...">Call</span>
      <svg class="group-hover/edit:translate-x-0.5 group-hover/edit:text-gray-500 ..."><!-- ... --></svg>
    </a>
  </li>
  {/each}
</ul>
```

对于内嵌的hover可以采用命名group实现

#### <a name="quick-start">🔋 兄弟元素状态改变如何影响</a>

```html
<form>
  <label class="block">
    <span class="...">Email</span>
    <input type="email" class="peer" />
    <!-- 如何input输入不是有效邮箱 下面则会visible-->
    <p class="invisible peer-invalid:visible">Please provide a valid email address.</p>
  </label>
</form>
```

注意peer-focus、peer-disabled同样适用

#### <a name="snippets">🔋 子元素根据父容器宽度展示</a>

```html
<div class="@container">
  <div class="flex flex-col @md:flex-row"></div>
  <!-- @min-[300px]当父容器宽度超过300时 -->
  <div class="flex-col @min-[300px]:flex-row"></div>
</div>
```

当父元素的宽度超过md时，展示为flex-row

#### <a name="links">🔗 根据js变量动态展示类名</a>

```html
<!-- 这种写法不生效 -->
<div class="text-{{ error ? 'red' : 'green' }}-600"></div>
<!-- works -->
<div class="{{ error ? 'text-red-600' : 'text-green-600' }}"></div>
```

```jsx
/** 这种写法不生效 */
function Button({ color, children }) {
  return <button className={`bg-${color}-600 hover:bg-${color}-500 ...`}>{children}</button>;
}

function Button({ color, children }) {
  const colorVariants = {
    blue: 'bg-blue-600 hover:bg-blue-500',
    red: 'bg-red-600 hover:bg-red-500',
  };
  return <button className={`${colorVariants[color]} ...`}>{children}</button>;
}
```

tailwind引擎不会识别变量字符串类名

#### <a name="more">🚀 使用utility自定义类名</a>

```css
@theme {
  /* 定义一个名为 --opacity-glass 的变量 */
  --opacity-glass: 15%;
  --opacity-heavy: 0.85;
}

@utility opacity-* {
  /* 
     1. 处理整数值的情况：
     如果捕获的值是整数（integer），将其乘以 1% 转换为百分比。
     例如：opacity-50 -> 捕获 50 -> calc(50 * 1%) -> 50% (即 0.5)
  */
  opacity: calc(--value(integer) * 1%);

  /* 
     2. 处理主题变量和任意值的情况：
     如果不是整数，尝试查找以 --opacity- 开头的主题变量，
     或者接受一个 [percentage] 类型的任意值。
  */
  opacity: --value(--opacity-*, [percentage]);
}
```

```html
<!-- 生成 CSS: opacity: 50%; -->
<div class="opacity-50">半透明</div>
<!-- 生成 CSS: opacity: 15%; -->
<div class="opacity-glass">毛玻璃效果</div>

<!-- 生成 CSS: opacity: 33.3%; -->
<div class="opacity-[33.3%]">精确透明度</div>
```

#### <a name="variant">🚀 使用变体动态编写样式</a>

```jsx
import { cva } from 'class-variance-authority';
import clsx from 'clsx';

const cardVariants = cva(
  // 1. 基础样式
  'flex items-center justify-center',
  {
    variants: {
      variant: {
        default: 'bg-slate-900 text-white hover:bg-slate-800',
        outline: 'border border-slate-200 bg-transparent hover:bg-slate-100',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 rounded-md px-3',
        lg: 'h-11 rounded-md px-8',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  },
);

export const Card = forwardRef<HTMLDivElement, CardProps>(
  ({ className, variant, size, ...props }, ref) => {
    return (
      <div
        ref={ref}
        className={cn(cardVariants({ variant, size, className }))}
        {...props}
      />
    );
  }
);

// 在页面使用
<Card variant={'outline'} size={'lg'} className="custom">
```
