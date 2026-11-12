# 前端技术分享：从 Tailwind CSS 到 shadcn/ui

> 主题：用原子化 CSS 打底，用"可拥有的组件"提效
> 适用对象：前端开发、设计协作同学
> 技术栈参考：React + Tailwind CSS v4

---

## 目录

1. [Tailwind CSS 是什么](#一tailwind-css-是什么)
2. [Tailwind CSS v4 的核心变化](#二tailwind-css-v4-的核心变化)
3. [Tailwind 的自定义能力：对组件封装与主题一致化的影响](#三tailwind-的自定义能力对组件封装与主题一致化的影响)
4. [Tailwind 的痛点：为什么还需要组件层](#四tailwind-的痛点为什么还需要组件层)
5. [shadcn/ui 是什么](#五shadcnui-是什么)
6. [shadcn/ui 安装与使用](#六shadcnui-安装与使用)
7. [shadcn/ui 的核心优势](#七shadcnui-的核心优势)
8. [应用场景与选型建议](#八应用场景与选型建议)
9. [最佳实践与注意事项](#九最佳实践与注意事项)
10. [总结](#十总结)

---

## 一、Tailwind CSS 是什么

Tailwind CSS 是一个 **Utility-First（工具类优先）** 的 CSS 框架。它不提供"按钮""卡片"这类成品组件，而是提供大量单一职责的原子类，让你直接在标记里组合出样式。

### 1.1 传统写法 vs Tailwind 写法

传统写法需要先起类名，再去 CSS 文件里写样式：

```html
<button class="btn-primary">提交</button>

<style>
  .btn-primary {
    padding: 8px 16px;
    border-radius: 6px;
    background: #2563eb;
    color: #fff;
  }
  .btn-primary:hover {
    background: #1d4ed8;
  }
</style>
```

Tailwind 写法直接在元素上组合：

```html
<button class="rounded-md bg-blue-600 px-4 py-2 text-white hover:bg-blue-700">
  提交
</button>
```

### 1.2 Tailwind 的优势

先看一张总览表，下面再逐条展开：

| 优势           | 一句话说明                                                      |
| -------------- | --------------------------------------------------------------- |
| 不用起类名     | 省去 `.wrapper-inner-box` 这类命名纠结                          |
| 样式就近       | 看结构就能看到样式，改动不用在文件间跳转                        |
| 产物体积小     | 只生成实际用到的类，CSS 体积不随页面数量线性增长                |
| 约束即规范     | 间距、字号、颜色都来自统一的设计刻度，天然减少"魔法数字"        |
| 响应式与状态   | `md:`、`hover:`、`focus-visible:`、`dark:` 等变体直接写在类名上 |
| 没有样式污染   | 每个类只做一件事，不存在选择器优先级大战和全局样式互相覆盖      |
| 删除代码无负担 | 删掉组件就删掉了它的样式，不会留下"没人敢删"的全局 CSS          |
| 高度可扩展     | 设计令牌、工具类、变体都能自定义，可以长成团队自己的样式语言    |

**1）开发效率：不切文件、不起名字**

写样式最耗时间的往往不是写属性，而是"给这个 div 起什么名字"和"这个类在哪个文件里"。Tailwind 把样式直接写在结构上，改一个间距只需要改一处，所见即所得。配合编辑器插件（Tailwind CSS IntelliSense）还有自动补全、悬停预览实际 CSS、颜色色块提示。

**2）可维护性：样式与组件同生共死**

传统 CSS 项目越做越大后，最常见的问题是"这个类还有没有地方在用？不敢删"。Tailwind 的样式写在组件内部，组件被删，样式也就跟着消失，不会产生死代码。重构时也不用担心改一个公共类影响到别的页面。

**3）性能：CSS 体积趋于稳定**

Tailwind 在构建时扫描源码，只输出实际出现过的类。由于工具类高度复用，项目页面从 10 个增长到 100 个时，CSS 体积增长非常有限，通常稳定在几十 KB（gzip 后更小）。

**4）一致性：设计刻度天然约束**

`p-4` 永远是 `1rem`，`text-sm` 永远是同一套字号和行高。开发者在一套有限的刻度中做选择，而不是随手写 `13px`、`15px`、`17px`，界面的节奏感自然统一。这一点在第三章"自定义能力"中会展开说明它对主题一致化的意义。

**5）表达力：状态与响应式一行写完**

hover、focus、disabled、暗色模式、断点、父元素状态、兄弟元素状态，都可以通过变体前缀写在类名上，不需要额外写选择器：

```html
<button
  class="bg-brand hover:bg-brand-hover focus-visible:ring-2 disabled:opacity-50 md:px-6"
>
  提交
</button>
```

**6）团队协作：降低代码风格差异**

不同开发者写出的 CSS 结构差异很大，而 Tailwind 的写法高度统一。配合 `prettier-plugin-tailwindcss` 自动排序类名、`eslint-plugin-tailwindcss` 检查不规范写法，Code Review 时几乎不用讨论样式风格问题。

### 1.3 常用语法速览

```html
<!-- 响应式：移动端单列，md 以上两列 -->
<div class="grid grid-cols-1 gap-4 md:grid-cols-2">...</div>

<!-- 状态变体 -->
<input
  class="border focus-visible:ring-2 focus-visible:ring-blue-500 disabled:opacity-50"
/>

<!-- 暗色模式 -->
<div class="bg-white text-gray-900 dark:bg-gray-900 dark:text-gray-100">
  ...
</div>

<!-- 任意值：只在刻度上没有对应值时使用，比如 calc() -->
<div class="top-[calc(100%+8px)]">...</div>

<!-- 父子联动（详见 1.4） -->
<a class="group">
  <span class="group-hover:underline">查看详情</span>
</a>
```

### 1.4 后代与兄弟元素语法

在传统 CSS 中，"父元素 hover 时改变子元素""输入框校验失败时显示提示文字"需要写后代选择器或兄弟选择器。Tailwind 把这些关系也做成了变体前缀，依然写在元素自己的 `class` 上。

#### 1.4.1 根据父元素/祖先元素的状态改样式：`group` 与 `in-*`

给父元素加 `group`，子元素用 `group-{状态}:` 响应父元素的状态：

```html
<a href="#" class="group rounded-lg p-4 hover:bg-gray-50">
  <h3 class="text-gray-900 group-hover:text-blue-600">标题</h3>
  <p class="text-gray-500 group-hover:text-gray-700">描述文字</p>
  <span class="opacity-0 transition group-hover:opacity-100">→</span>
</a>
```

对应的 CSS 大致是 `.group:hover .group-hover\:text-blue-600 { ... }`。所有状态都能和 `group` 组合：`group-focus-within:`、`group-active:`、`group-disabled:`、`group-data-[state=open]:`、`group-aria-expanded:` 等。

**嵌套分组时用命名 group**，避免内层误响应外层：

```html
<ul class="group/list">
  <li class="group/item flex justify-between">
    <span>项目名称</span>
    <!-- 只在 hover 当前 li 时出现，而不是 hover 整个列表时 -->
    <button class="invisible group-hover/item:visible">编辑</button>
  </li>
</ul>
```

**不想加 `group` 类时用 `in-*`**：匹配任意祖先元素的状态，写法更短：

```html
<div tabindex="0">
  <span class="opacity-50 in-focus:opacity-100">祖先获得焦点时高亮</span>
</div>
```

#### 1.4.2 父元素控制子元素：`*:`、`**:` 与任意变体

有时样式要统一施加给一批子元素，又不想在每个子元素上重复写类：

```html
<!-- *: 作用于所有直接子元素 -->
<ul class="flex gap-2 *:rounded-full *:bg-gray-100 *:px-3 *:py-1">
  <li>React</li>
  <li>Vue</li>
  <li>Svelte</li>
</ul>

<!-- **: 作用于所有后代元素 -->
<div class="**:text-sm">...</div>
```

更精确的控制可以用**任意变体** `[选择器]:`，`&` 代表当前元素，`_` 代表空格：

| 写法                              | 对应 CSS 选择器           | 含义                         |
| --------------------------------- | ------------------------- | ---------------------------- |
| `[&>li]:mt-2`                     | `.x > li`                 | 直接子元素中的 `li`          |
| `[&_p]:leading-7`                 | `.x p`                    | 所有后代 `p`（`_` 表示空格） |
| `[&_a:hover]:underline`           | `.x a:hover`              | 后代链接 hover 时            |
| `[&>*:not(:last-child)]:border-b` | `.x > *:not(:last-child)` | 除最后一个外的直接子元素     |

这种写法特别适合渲染富文本、Markdown 等**无法给内部元素加类名**的场景。

#### 1.4.3 根据子元素的状态改父元素：`has-*`

CSS 的 `:has()` 让"父选择器"成为可能，Tailwind 对应 `has-*` 变体：

```html
<!-- 内部的单选框被选中时，整张卡片高亮 -->
<label
  class="rounded-lg border p-4 has-checked:border-blue-600 has-checked:bg-blue-50"
>
  <input type="radio" name="plan" class="sr-only" />
  专业版
</label>

<!-- 内部有图片时改变布局 -->
<div class="grid has-[img]:grid-cols-2">...</div>
```

`has-*` 也能和 group / peer 组合：`group-has-[:invalid]:`、`peer-has-checked:`。

#### 1.4.4 根据兄弟元素的状态改样式：`peer`

给前面的兄弟元素加 `peer`，后面的兄弟元素用 `peer-{状态}:` 响应它：

```html
<label>
  <input
    type="email"
    class="peer rounded border px-3 py-2 invalid:border-red-500"
  />
  <p class="invisible text-sm text-red-500 peer-invalid:visible">
    请输入正确的邮箱地址
  </p>
</label>
```

对应的 CSS 是 `.peer:invalid ~ .peer-invalid\:visible { ... }`。

**注意：`peer` 只能影响它后面的兄弟元素**，这是 CSS 通用兄弟选择器 `~` 的限制。如果要被影响的元素在前面，需要调整 DOM 顺序（必要时再用 `order-*` 或 `flex-row-reverse` 调回视觉顺序）。

同样支持命名，适合同一容器里有多个 peer 的情况：

```html
<input id="email" class="peer/email" />
<input id="password" class="peer/password" />
<p class="hidden peer-invalid/email:block">邮箱格式错误</p>
<p class="hidden peer-invalid/password:block">密码格式错误</p>
```

常见的 `peer` 用法：浮动标签（`peer-placeholder-shown:`）、自定义复选框/开关（`peer-checked:`）、表单校验提示（`peer-invalid:`）。

#### 1.4.5 根据位置选择兄弟元素：结构伪类

| 变体                       | 对应 CSS                             | 常见用途                 |
| -------------------------- | ------------------------------------ | ------------------------ |
| `first:` / `last:`         | `:first-child` / `:last-child`       | 首尾元素去掉边框、间距   |
| `odd:` / `even:`           | `:nth-child(odd/even)`               | 表格斑马纹               |
| `nth-3:` / `nth-[3n+1]:`   | `:nth-child(3)` / `:nth-child(3n+1)` | 指定位置的元素           |
| `only:`                    | `:only-child`                        | 只有一个子元素时         |
| `empty:`                   | `:empty`                             | 空状态时隐藏             |
| `not-last:` / `not-first:` | `:not(:last-child)` 等               | "除了最后一个都加分隔线" |

```html
<ul>
  <li class="py-3 not-last:border-b">第一项</li>
  <li class="py-3 not-last:border-b">第二项</li>
  <li class="py-3 not-last:border-b">第三项</li>
</ul>

<!-- 也可以在父元素上统一处理 -->
<ul class="divide-y">
  ...
</ul>
<tbody class="*:odd:bg-gray-50">
  ...
</tbody>
```

> 布局间距优先用 `flex` / `grid` 的 `gap-*`。`space-x-*` / `space-y-*` 本质上是给兄弟元素加 margin，遇到换行、隐藏元素时容易出问题。

#### 1.4.6 速查

| 需求                    | 写法                                   |
| ----------------------- | -------------------------------------- |
| 父元素状态 → 子元素样式 | 父 `group`，子 `group-hover:`          |
| 任意祖先状态 → 当前元素 | `in-focus:`、`in-data-[state=open]:`   |
| 父元素统一控制子元素    | `*:`、`**:`、`[&>li]:`、`[&_p]:`       |
| 子元素状态 → 父元素样式 | `has-checked:`、`has-[img]:`           |
| 前面兄弟状态 → 后面兄弟 | 前 `peer`，后 `peer-invalid:`          |
| 按位置选择              | `first:`、`last:`、`odd:`、`not-last:` |

### 1.5 响应式断点：`sm`、`md`、`lg`……

#### 1.5.1 默认断点

| 前缀   | 最小宽度        | 对应 CSS                      |
| ------ | --------------- | ----------------------------- |
| `sm:`  | 40rem（640px）  | `@media (width >= 40rem) { }` |
| `md:`  | 48rem（768px）  | `@media (width >= 48rem) { }` |
| `lg:`  | 64rem（1024px） | `@media (width >= 64rem) { }` |
| `xl:`  | 80rem（1280px） | `@media (width >= 80rem) { }` |
| `2xl:` | 96rem（1536px） | `@media (width >= 96rem) { }` |

#### 1.5.2 移动优先（Mobile First）

**不带前缀的类对所有屏幕生效，带前缀的类表示"从这个宽度开始及以上"生效**：

```html
<!-- 手机 1 列，≥768px 2 列，≥1024px 4 列 -->
<div class="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-4">...</div>

<!-- 字号随屏幕逐级放大 -->
<h1 class="text-2xl md:text-4xl xl:text-5xl">标题</h1>

<!-- 手机端隐藏，≥768px 显示 -->
<nav class="hidden md:flex">...</nav>
```

常见误区：`sm:` **不是**"手机端"的意思。想写手机端样式，直接写不带前缀的类即可，再用 `md:` 等去覆盖大屏。

#### 1.5.3 最大宽度与区间

```html
<!-- max-*：小于该断点时生效，这里是 < 768px -->
<div class="max-md:text-center">...</div>

<!-- 区间：仅在 768px ≤ 宽度 < 1280px 时生效 -->
<div class="md:max-xl:px-8">...</div>

<!-- 只在 md 这一档生效 -->
<div class="md:max-lg:hidden">...</div>
```

#### 1.5.4 一次性断点：任意值

设计稿里偶尔出现某个特殊宽度、又不值得加进全局断点时：

```html
<div class="min-[900px]:flex max-[480px]:hidden">...</div>
```

#### 1.5.5 自定义、新增、删除断点

在 `@theme` 中通过 `--breakpoint-*` 命名空间管理：

```css
@import "tailwindcss";

@theme {
  /* 新增 */
  --breakpoint-xs: 30rem; /* 480px → xs: */
  --breakpoint-3xl: 120rem; /* 1920px → 3xl: */

  /* 覆盖默认值 */
  --breakpoint-2xl: 90rem; /* 1440px */
}
```

完全替换为团队自己的断点体系：

```css
@theme {
  --breakpoint-*: initial; /* 清空全部默认断点 */
  --breakpoint-tablet: 48rem;
  --breakpoint-laptop: 64rem;
  --breakpoint-desktop: 90rem;
}
```

```html
<div class="grid-cols-1 tablet:grid-cols-2 desktop:grid-cols-4">...</div>
```

> 断点建议统一使用 `rem`，不要 `px` 与 `rem` 混用。Tailwind 会按数值对断点排序，单位不一致可能导致生成顺序错误、覆盖关系异常。

#### 1.5.6 容器查询：按父容器宽度响应

断点看的是**视口宽度**，而组件放在侧边栏和放在主内容区时，可用宽度并不同。容器查询让组件根据**父容器宽度**调整布局，更适合做可复用组件：

```html
<div class="@container">
  <div class="flex flex-col @md:flex-row @xl:gap-8">...</div>
</div>

<!-- 命名容器 + 区间 + 任意值 -->
<div class="@container/card">
  <div class="@sm/card:grid-cols-2 @max-md:text-sm @min-[30rem]:p-6">...</div>
</div>
```

容器查询的尺寸来自 `--container-*` 命名空间（`@3xs` 16rem 到 `@7xl` 80rem），新增方式与断点相同：`--container-card: 30rem;` 之后即可写 `@card:`。

### 1.6 尺寸体系：如何使用、定义和新增尺寸

#### 1.6.1 间距刻度的规则

Tailwind 的 `p-*`、`m-*`、`gap-*`、`w-*`、`h-*`、`size-*`、`inset-*`、`top-*` 等都基于一个基础单位：

```css
@theme {
  --spacing: 0.25rem; /* 默认值，1 个单位 = 4px */
}
```

工具类数字 × 基础单位 = 实际尺寸，即 **`N px` 对应 `N ÷ 4`**：

| 设计稿 | 工具类      | 实际值   |
| ------ | ----------- | -------- |
| 4px    | `p-1`       | 0.25rem  |
| 10px   | `p-2.5`     | 0.625rem |
| 18px   | `mt-4.5`    | 1.125rem |
| 52px   | `h-13`      | 3.25rem  |
| 300px  | `w-75`      | 18.75rem |
| 1200px | `max-w-300` | 75rem    |

v4 中**任意数字都可以直接用**（只要是 0.25 的倍数），`w-75`、`h-13`、`gap-4.5` 无需任何配置即可生效。v3 只能用预设好的那几档，这是 v4 很实用的一个改进。

另外还有几个固定写法：`w-px`（1px）、`w-full`、`w-screen`、`w-1/2`、`min-h-dvh`、`size-10`（宽高同时设为 2.5rem）。

#### 1.6.2 新增具名尺寸：`--spacing-*`

当某个尺寸有明确的业务含义（头部高度、侧边栏宽度）时，给它起名字比记数字更好：

```css
@theme {
  --spacing-header: 4rem; /* 64px */
  --spacing-sidebar: 17.5rem; /* 280px */
}
```

所有间距类工具都能直接使用：

```html
<header class="fixed h-header">...</header>
<aside class="w-sidebar">...</aside>
<main class="pt-header lg:pl-sidebar">...</main>
<section class="scroll-mt-header">...</section>
```

以后头部高度从 64px 改成 72px，只改一行变量，所有引用同步更新。

#### 1.6.3 定义其他类型的尺寸

不同类型的尺寸各有自己的命名空间：

```css
@theme {
  /* 字号：可同时绑定行高、字重、字间距 */
  --text-display: 3.5rem; /* 56px → text-display */
  --text-display--line-height: 1.2;
  --text-display--font-weight: 700;
  --text-display--letter-spacing: -0.02em;

  /* 圆角 */
  --radius-card: 0.75rem; /* 12px → rounded-card */

  /* 最大宽度（同时也是容器查询尺寸） */
  --container-content: 75rem; /* 1200px → max-w-content */

  /* 行高、字间距 */
  --leading-loose-plus: 2.2; /* leading-loose-plus */
  --tracking-wide-plus: 0.08em; /* tracking-wide-plus */
}
```

| 命名空间         | 作用          | 生成的类名                   |
| ---------------- | ------------- | ---------------------------- |
| `--spacing`      | 间距基础单位  | `p-4`、`w-75` 等全部间距类   |
| `--spacing-*`    | 具名间距/宽高 | `h-header`、`pt-header`      |
| `--text-*`       | 字号          | `text-display`               |
| `--radius-*`     | 圆角          | `rounded-card`               |
| `--container-*`  | 容器尺寸      | `max-w-content`、`@content:` |
| `--breakpoint-*` | 断点          | `3xl:`                       |
| `--leading-*`    | 行高          | `leading-loose-plus`         |
| `--tracking-*`   | 字间距        | `tracking-wide-plus`         |

想彻底替换某一类默认刻度（比如只允许设计规范中的 6 个字号），先清空再定义：

```css
@theme {
  --text-*: initial;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  /* ... */
}
```

#### 1.6.4 修改基础单位

如果设计稿以 8px 为网格，也可以直接修改基础单位：

```css
@theme {
  --spacing: 0.5rem; /* 1 个单位 = 8px，p-2 = 16px */
}
```

这会改变**所有**间距类的含义，适合新项目一开始就定下来；已有项目中途修改会导致全站间距变化，需谨慎。

#### 1.6.5 一次性尺寸：任意值与 CSS 变量

只出现一次、或者需要计算的尺寸，用任意值或 CSS 变量：

```html
<!-- 计算值 -->
<div class="h-[calc(100dvh-4rem)]">...</div>

<!-- 引用 CSS 变量：v4 的 (--var) 简写 -->
<div class="w-(--sidebar-width)">...</div>
```

#### 1.6.6 尺寸选择优先级

1. **默认刻度**：能整除 4px 的，直接用 `p-6`、`w-75`。
2. **具名令牌**：有业务含义、多处复用的，用 `--spacing-*`、`--text-*` 等定义。
3. **任意值**：`calc()`、CSS 变量、确实无法落到刻度上的一次性值，才用 `[...]`。

> 令牌建议用 `rem` 定义、在注释里标注对应的 `px`。`rem` 会跟随用户浏览器的字号设置缩放，对无障碍更友好。

---

## 二、Tailwind CSS v4 的核心变化

v4 是一次大版本重构，和 v3 的差异主要在配置方式上。

### 2.1 CSS-first 配置，告别 `tailwind.config.js`

v3 在 JS 文件里配置主题，v4 直接在 CSS 里用 `@theme` 声明：

```css
/* app.css */
@import "tailwindcss";

@theme {
  --color-brand: oklch(0.62 0.19 255);
  --font-display: "Inter", sans-serif;
  --breakpoint-3xl: 1920px;
}
```

声明后会自动生成对应的工具类：`bg-brand`、`text-brand`、`font-display`、`3xl:` 等。

### 2.2 其他关键变化

- **安装更简单**：PostCSS 场景只需 `@tailwindcss/postcss`，入口一行 `@import "tailwindcss"`。
- **自动内容扫描**：不再需要手动配置 `content` 路径。
- **主题即 CSS 变量**：`@theme` 中的值会输出为原生 CSS 变量，运行时可直接 `var(--color-brand)` 使用，这一点是后面 shadcn/ui 主题方案的基础。
- **构建性能大幅提升**：新引擎全量构建和增量构建都明显更快。
- **现代 CSS 特性**：原生支持容器查询（`@container`）、`oklch` 色彩空间、级联层（`@layer`）等。

---

## 三、Tailwind 的自定义能力：对组件封装与主题一致化的影响

Tailwind 真正强大的地方不是它自带的那几千个类，而是**可以把团队的设计规范变成新的类名、新的属性、新的变体**。这套机制直接决定了组件怎么封装、主题怎么统一。

### 3.1 自定义设计令牌：`@theme` 生成团队专属类名

在 `@theme` 中声明的变量会按命名空间自动生成对应工具类：

| 命名空间         | 示例变量             | 生成的类名                               |
| ---------------- | -------------------- | ---------------------------------------- |
| `--color-*`      | `--color-brand`      | `bg-brand`、`text-brand`、`border-brand` |
| `--text-*`       | `--text-title`       | `text-title`                             |
| `--leading-*`    | `--leading-relaxed`  | `leading-relaxed`                        |
| `--font-*`       | `--font-display`     | `font-display`                           |
| `--radius-*`     | `--radius-card`      | `rounded-card`                           |
| `--shadow-*`     | `--shadow-panel`     | `shadow-panel`                           |
| `--breakpoint-*` | `--breakpoint-3xl`   | `3xl:` 响应式前缀                        |
| `--spacing`      | `--spacing: 0.25rem` | 所有 `p-*`、`m-*`、`w-*` 的基础单位      |

一个典型的品牌令牌定义：

```css
:root {
  --brand: #2563eb;
  --brand-hover: #1d4ed8;
}

@theme inline {
  --color-brand: var(--brand);
  --color-brand-hover: var(--brand-hover);

  --text-title: 1.375rem; /* 22px */
  --text-title--line-height: 1.4;
  --radius-card: 0.75rem; /* 12px */
}
```

之后所有组件写的都是 `bg-brand`、`hover:bg-brand-hover`、`text-title`、`rounded-card`，而不是 `bg-[#2563eb]`、`text-[22px]`、`rounded-[12px]`。

**对主题一致化的影响：**

1. **单一数据源**：品牌色、字号、圆角只在一个地方定义，组件里只引用名字。改品牌色只改 `--brand` 一行，全站生效。
2. **语义取代数值**：`bg-brand` 表达的是"品牌主色"，`bg-[#2563eb]` 只是一个颜色值。语义化命名让代码能直接对应到设计规范，设计改版时也能清楚知道哪些地方受影响。
3. **收紧可选范围**：可以用 `--color-*: initial;` 清空默认调色板，只保留团队定义的颜色。这样开发者根本写不出规范外的颜色，一致性从"靠自觉"变成"靠约束"。
4. **与设计工具对齐**：设计稿里的 Figma 变量名可以和 `@theme` 中的令牌一一对应，设计和开发说同一种语言。

> 建议规则：刻度上能表达的值一律用令牌或默认刻度类，任意值 `[...]` 只留给 `calc()`、CSS 变量等确实无法落到刻度上的场景，可以借助 `eslint-plugin-tailwindcss` 来检查。

### 3.2 自定义工具类：`@utility` 沉淀可复用的样式模式

一些跨组件重复出现、又不属于设计令牌的样式组合，可以注册成新的工具类。它和原生工具类一样支持所有变体：

```css
/* 页面内容区的统一水平留白 */
@utility page-inline {
  padding-inline: max(1.5rem, calc((100% - 75rem) / 2));
}

/* 隐藏滚动条 */
@utility scrollbar-none {
  scrollbar-width: none;
  &::-webkit-scrollbar {
    display: none;
  }
}

/* 带参数的函数式工具类：line-clamp-safe-2、line-clamp-safe-3 ... */
@utility line-clamp-safe-* {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: --value(integer);
  overflow: hidden;
}
```

```html
<section class="page-inline md:py-20">...</section>
<ul class="scrollbar-none overflow-x-auto">
  ...
</ul>
<p class="line-clamp-safe-2">...</p>
```

**对组件封装的影响：** 把"约定俗成"的写法升级成"有名字的规范"。比起在每个组件里复制同一段 `px-[max(...)]`，一个有名字的类更容易统一修改，也更容易在 Code Review 中被识别出来。

> 与 `@apply` 的区别：`@apply` 是把一组工具类塞进自定义选择器（如 `.btn { @apply px-4 py-2 }`），容易退回到传统 CSS 的命名和覆盖问题；`@utility` 产出的仍是原子类，可以与变体组合、可以被 `tailwind-merge` 正确处理，推荐优先使用。

### 3.3 自定义变体：`@custom-variant` 定义主题作用域

变体决定样式"在什么条件下生效"。除了内置的 `hover:`、`dark:`，还可以自定义：

```css
/* 用类名控制暗色模式，而不是只跟随系统 */
@custom-variant dark (&:where(.dark, .dark *));

/* 某个区域切换为深色头部主题 */
@custom-variant on-dark-header (&:where([data-header="dark"] *));
```

```html
<header data-header="dark">
  <a class="text-foreground on-dark-header:text-white">首页</a>
</header>
```

**对主题一致化的影响：** 主题切换的"开关"被统一定义在 CSS 层，组件只声明"在该主题下长什么样"，不需要在 JS 中判断当前主题再拼类名。

### 3.4 属性驱动样式：用 `data-*` / `aria-*` 连接组件状态与样式

Tailwind 支持直接根据元素属性写样式，这是组件封装中最重要的一个能力：

```tsx
<button
  data-state={open ? "open" : "closed"}
  aria-expanded={open}
  className="text-foreground data-[state=open]:text-brand aria-expanded:font-medium"
>
  产品
  <ChevronDown className="transition-transform in-data-[state=open]:rotate-180" />
</button>
```

常用的属性与关系类变体：

| 变体                                | 作用                                 |
| ----------------------------------- | ------------------------------------ |
| `data-[state=open]:`                | 匹配 `data-state="open"`             |
| `data-active:`                      | 匹配存在 `data-active` 属性          |
| `aria-selected:` / `aria-disabled:` | 匹配 ARIA 状态                       |
| `group-data-[state=open]:`          | 父元素（`group`）处于某状态时        |
| `peer-checked:` / `peer-invalid:`   | 前一个兄弟元素（`peer`）处于某状态时 |
| `has-[:checked]:`                   | 自身包含被选中的子元素时             |
| `in-data-[state=open]:`             | 任意祖先元素处于某状态时             |
| `*:`                                | 作用于所有直接子元素                 |

**对组件封装的影响：**

1. **逻辑和样式解耦**：组件逻辑只负责输出状态属性（`data-state`、`aria-expanded`），样式根据属性变化。不再需要 `className={open ? "a b c" : "d e f"}` 这种三元拼接。
2. **无障碍与样式同源**：样式直接挂在 `aria-*` 上，只要无障碍属性写对了，视觉状态就一定正确；反过来也倒逼开发者把 ARIA 写完整。
3. **外部可定制**：使用方拿到组件后，可以通过 `className="data-[state=open]:bg-muted"` 定制任意状态的样式，组件不需要为每种状态额外暴露 prop。
4. **与 Headless 组件天然契合**：Radix UI、Base UI 等无样式组件库都会输出 `data-state`、`data-side`、`data-disabled` 等属性，shadcn/ui 正是基于这一点，用纯 Tailwind 类完成全部交互状态样式。

### 3.5 CSS 变量作为组件参数：把"可调的部分"暴露出去

Tailwind v4 可以用 `(--var)` 简写直接读取 CSS 变量，也可以用任意属性 `[--var:value]` 在元素上设置变量：

```tsx
function Card({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      className={cn(
        "rounded-lg bg-background p-(--card-padding) [--card-padding:--spacing(6)]",
        className,
      )}
      {...props}
    />
  );
}

// 使用方只改变量，不需要知道内部用了哪些类
<Card className="[--card-padding:--spacing(4)]" />;
```

**对组件封装的影响：** 组件可以把"允许外部调整的部分"以 CSS 变量的形式暴露，内部结构保持稳定。同时这些变量可以在父容器上统一设置，向下继承，适合批量调整一组组件（比如紧凑模式下统一缩小内边距）。

### 3.6 小结：自定义能力如何塑造组件与主题

| 能力              | 对组件封装的影响                     | 对主题一致化的影响                     |
| ----------------- | ------------------------------------ | -------------------------------------- |
| `@theme` 令牌     | 组件只引用语义类名，不依赖具体数值   | 单一数据源，改一处全站生效             |
| `@utility`        | 复用的样式模式有了名字，便于统一修改 | 跨组件的布局与视觉规则保持一致         |
| `@custom-variant` | 组件声明各主题下的样式，无需 JS 判断 | 主题开关集中在 CSS 层定义              |
| `data-*`/`aria-*` | 逻辑输出状态，样式响应状态，彼此解耦 | 所有组件的交互状态表现一致             |
| CSS 变量参数      | 组件暴露可调参数，内部结构稳定       | 父级统一设置，实现密度、尺寸等全局切换 |

这五种能力组合起来，就是一套完整的"设计系统基础设施"。**shadcn/ui 本质上就是在这套基础设施上做好的一组组件**：它用语义令牌做主题、用 `data-state` 做状态样式、用 CSS 变量做可调参数。理解了本章，后面 shadcn/ui 的设计思路就很好理解了。

---

## 四、Tailwind 的痛点：为什么还需要组件层

Tailwind 解决了"怎么写样式"，但没有解决"怎么写组件"：

1. **类名冗长重复**：一个按钮十几个类，项目里复制粘贴几十份，改一处要改全部。
2. **交互与无障碍要自己做**：下拉菜单、弹窗、Tooltip、Combobox 涉及焦点管理、键盘导航、ARIA 属性、点击外部关闭、滚动锁定，这些都不是 CSS 能解决的。
3. **变体管理混乱**：按钮有 `primary / secondary / ghost` × `sm / md / lg` × `disabled / loading`，用字符串拼接很快失控。
4. **类名冲突**：外部传入 `className="px-2"` 想覆盖内部的 `px-4`，结果两个类都生效，最终效果取决于 CSS 生成顺序。

传统组件库（Ant Design、MUI、Element Plus）能解决前两点，但又带来新问题：样式难以深度定制、和 Tailwind 风格割裂、升级受制于库的版本、包体积大。

**shadcn/ui 的定位正好卡在中间：既有成熟的组件能力，又保持 Tailwind 的完全可控。**

---

## 五、shadcn/ui 是什么

官方的一句话定位：

> This is not a component library. It is how you build your component library.
> 它不是一个组件库，而是你构建自己组件库的方式。

### 5.1 和传统组件库的本质区别

|          | 传统组件库（Ant Design / MUI） | shadcn/ui                      |
| -------- | ------------------------------ | ------------------------------ |
| 分发方式 | npm 包，代码在 `node_modules`  | CLI 把**源码复制**到你的项目里 |
| 代码归属 | 属于库作者                     | 属于你，可以随意修改           |
| 样式方案 | CSS-in-JS / Less / 自有体系    | Tailwind CSS + CSS 变量        |
| 定制方式 | 主题配置 / 样式覆盖            | 直接改源码                     |
| 升级方式 | 升级依赖版本                   | 按需重新拉取或手动合并         |
| 打包体积 | 依赖整体或依赖 Tree-shaking    | 只有你加进来的组件             |

### 5.2 技术构成

shadcn/ui 本身是对几样成熟技术的组合：

- **无样式交互原语（Headless Primitives）**：Radix UI（或 Base UI），负责焦点管理、键盘交互、ARIA、Portal 等"难而无聊"的部分。
- **Tailwind CSS**：负责全部视觉样式。
- **class-variance-authority（cva）**：用声明式方式管理组件变体。
- **clsx + tailwind-merge**：合并类名并自动解决 Tailwind 类冲突，封装为 `cn()` 工具函数。
- **CLI + Registry**：负责把组件源码和依赖安装到项目中。

---

## 六、shadcn/ui 安装与使用

### 6.1 初始化

在已经配置好 Tailwind 的 Next.js 项目中执行：

```bash
pnpm dlx shadcn@latest init
```

CLI 会询问基础色、样式风格等，然后完成以下事情：

- 生成 `components.json`（记录组件目录、别名、样式配置）
- 在全局 CSS 中写入主题 CSS 变量
- 创建 `lib/utils.ts`，提供 `cn()` 函数
- 安装 `clsx`、`tailwind-merge`、`class-variance-authority` 等依赖

`components.json` 示例：

```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "new-york",
  "rsc": true,
  "tsx": true,
  "tailwind": {
    "config": "",
    "css": "src/app/globals.css",
    "baseColor": "neutral",
    "cssVariables": true
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui",
    "lib": "@/lib",
    "hooks": "@/hooks"
  },
  "iconLibrary": "lucide"
}
```

> Tailwind v4 项目中 `tailwind.config` 留空即可，主题完全由 CSS 变量驱动。

### 6.2 添加组件

```bash
# 添加单个组件
pnpm dlx shadcn@latest add button

# 一次添加多个
pnpm dlx shadcn@latest add dialog dropdown-menu form input
```

执行后，组件源码会出现在 `src/components/ui/` 下，例如 `button.tsx`，同时自动安装它需要的依赖。

### 6.3 使用组件

```tsx
import { Button } from "@/components/ui/button";

export default function Page() {
  return (
    <div className="flex gap-2">
      <Button>默认按钮</Button>
      <Button variant="outline">描边按钮</Button>
      <Button variant="ghost" size="sm">
        幽灵按钮
      </Button>
      <Button asChild>
        <a href="/contact">渲染为链接</a>
      </Button>
    </div>
  );
}
```

### 6.4 读懂一个组件源码

以 `Button` 为例（节选），这就是你项目里实际拥有的代码：

```tsx
import { Slot } from "@radix-ui/react-slot";
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

const buttonVariants = cva(
  "inline-flex items-center justify-center gap-2 rounded-md text-sm font-medium transition-all disabled:pointer-events-none disabled:opacity-50 focus-visible:ring-[3px] focus-visible:ring-ring/50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive: "bg-destructive text-white hover:bg-destructive/90",
        outline:
          "border bg-background hover:bg-accent hover:text-accent-foreground",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        default: "h-9 px-4 py-2",
        sm: "h-8 px-3",
        lg: "h-10 px-6",
        icon: "size-9",
      },
    },
    defaultVariants: { variant: "default", size: "default" },
  },
);

function Button({
  className,
  variant,
  size,
  asChild = false,
  ...props
}: React.ComponentProps<"button"> &
  VariantProps<typeof buttonVariants> & { asChild?: boolean }) {
  const Comp = asChild ? Slot : "button";
  return (
    <Comp
      className={cn(buttonVariants({ variant, size, className }))}
      {...props}
    />
  );
}

export { Button, buttonVariants };
```

`cn()` 的实现只有几行：

```ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

`twMerge` 保证 `cn("px-4", "px-2")` 的结果是 `px-2`，外部传入的 `className` 一定能覆盖内部默认样式。

---

## 七、shadcn/ui 的核心优势

### 7.1 主题替换：改几个 CSS 变量，全站换肤

shadcn/ui 的所有组件都不直接写死颜色，而是使用**语义化的设计令牌**：`bg-primary`、`text-muted-foreground`、`border-border`、`ring-ring`。这些令牌最终映射到 CSS 变量。

```css
@import "tailwindcss";

:root {
  --radius: 0.625rem;
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
  --primary: oklch(0.205 0 0);
  --primary-foreground: oklch(0.985 0 0);
  --muted: oklch(0.97 0 0);
  --muted-foreground: oklch(0.556 0 0);
  --border: oklch(0.922 0 0);
  --ring: oklch(0.708 0 0);
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
  --primary: oklch(0.922 0 0);
  --primary-foreground: oklch(0.205 0 0);
  /* ... */
}

/* 把 CSS 变量注册为 Tailwind 工具类 */
@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-muted: var(--muted);
  --color-muted-foreground: var(--muted-foreground);
  --color-border: var(--border);
  --color-ring: var(--ring);
  --radius-md: calc(var(--radius) - 2px);
  --radius-lg: var(--radius);
}
```

这套机制带来的好处：

1. **品牌换色零侵入**：把 `--primary` 改成品牌色，所有按钮、链接、焦点环、选中态同步生效，组件代码一行不用动。
2. **暗色模式开箱即用**：在 `<html>` 上切换 `.dark` 类即可，配合 `next-themes` 还能跟随系统。
3. **多主题 / 多品牌**：再定义 `[data-theme="blue"]`、`[data-theme="green"]` 等选择器覆盖变量，就能实现 SaaS 多租户换肤、节日主题等需求。
4. **圆角、阴影统一调节**：`--radius` 一个变量控制全站圆角风格，从"方正商务风"到"圆润活泼风"只需改一个值。
5. **前景色成对出现**：`primary` / `primary-foreground` 的命名约定保证了背景和文字的对比度成对设计，降低出现可读性问题的概率。
6. **设计协作友好**：语义令牌可以直接与 Figma Variables 对齐，设计稿和代码说的是同一套语言。官方 Themes 页面也可以可视化调色后直接复制 CSS。

### 7.2 高自定义：从"覆盖样式"变成"修改源码"

传统组件库定制样式，经常要和库"打架"：

```css
/* 传统库常见写法：靠高优先级选择器硬覆盖 */
.ant-btn.ant-btn-primary:not(:disabled):hover {
  background: #xxx !important;
}
```

shadcn/ui 的定制方式有三个层次，由浅到深：

**层次一：调用时传 `className`（单点微调）**

```tsx
<Button className="h-12 rounded-full px-8">大号圆角按钮</Button>
```

得益于 `tailwind-merge`，传入的类会正确覆盖默认类，不会出现冲突。

**层次二：修改 `cva` 变体（全局新增风格）**

```tsx
const buttonVariants = cva("...", {
  variants: {
    variant: {
      // ...原有变体
      brand:
        "bg-gradient-to-r from-indigo-500 to-violet-500 text-white shadow-lg hover:opacity-90",
    },
    size: {
      // ...原有尺寸
      xl: "h-14 px-10 text-base",
    },
  },
});
```

```tsx
<Button variant="brand" size="xl">
  立即体验
</Button>
```

TypeScript 会自动推导出新的 `variant` 和 `size` 类型，调用处有完整提示。

**层次三：直接改结构和行为（深度定制）**

比如给按钮加 `loading` 状态，直接在源码里加：

```tsx
function Button({
  loading,
  children,
  disabled,
  ...props
}: ButtonProps & { loading?: boolean }) {
  return (
    <Comp disabled={disabled || loading} {...props}>
      {loading && <Loader2 className="size-4 animate-spin" />}
      {children}
    </Comp>
  );
}
```

不需要提 Issue 等官方支持，也不需要 fork 整个库。

**组合式 API 也是高自定义的基础**：shadcn/ui 的复杂组件都拆成了可组合的小块，可以自由增删、重排：

```tsx
<Dialog>
  <DialogTrigger asChild>
    <Button variant="outline">编辑资料</Button>
  </DialogTrigger>
  <DialogContent className="sm:max-w-md">
    <DialogHeader>
      <DialogTitle>编辑资料</DialogTitle>
      <DialogDescription>修改后点击保存。</DialogDescription>
    </DialogHeader>
    {/* 中间放任何内容 */}
    <DialogFooter>
      <Button type="submit">保存</Button>
    </DialogFooter>
  </DialogContent>
</Dialog>
```

### 7.3 完全可控：代码是你的，不是依赖

这是 shadcn/ui 和所有传统组件库最根本的区别。

1. **源码在仓库里**：组件就是 `src/components/ui/*.tsx`，可读、可改、可调试，出问题直接打断点，不用翻 `node_modules`。
2. **不受上游版本绑架**：没有"升级大版本导致 API 全改""某个 Bug 等官方修复""库停止维护"的风险。你决定什么时候、要不要跟进上游改动。
3. **没有黑盒行为**：组件内部用了什么 DOM 结构、什么 ARIA 属性、什么动画，一目了然，方便做 SEO、无障碍审查和性能优化。
4. **按需引入，体积可控**：只有执行过 `add` 的组件才会进入项目，不存在"为了一个按钮引入整个库"的情况。
5. **依赖透明**：每个组件依赖什么（Radix 某个包、`lucide-react` 等）都会显式写进 `package.json`，可审计、可替换。
6. **可沉淀为团队组件库**：在 shadcn 组件基础上改造出符合团队规范的版本，配合自建 Registry，团队内其他项目也能用 `shadcn add` 一键安装，形成真正属于自己的设计系统。

### 7.4 其他优势

- **无障碍做得扎实**：底层 Radix / Base UI 已处理好键盘导航、焦点陷阱、屏幕阅读器语义，这是自己手写最容易做漏的部分。
- **与 React 生态深度集成**：表单与 `react-hook-form` + `zod` 配合，表格基于 `@tanstack/react-table`，图表基于 Recharts，日期选择基于 `react-day-picker`，都是社区主流方案。
- **支持 React Server Components**：`components.json` 中 `rsc: true` 时，CLI 会自动为需要的组件加上 `"use client"`。
- **Blocks 区块模板**：官方提供登录页、仪表盘、侧边栏等完整区块，可直接 `add` 进项目作为起点。
- **生态繁荣**：大量第三方 Registry（动画组件、营销页区块等）兼容同一套 CLI 和主题变量，主题可以无缝共享。
- **对 AI 编码友好**：代码就在项目中，结构统一、约定清晰，AI 助手能直接读到组件源码并生成风格一致的代码；官方也提供 MCP Server，让 AI 工具可以检索和安装 Registry 中的组件。

---

## 八、应用场景与选型建议

### 8.1 非常适合的场景

| 场景                     | 原因                                                               |
| ------------------------ | ------------------------------------------------------------------ |
| 中后台 / 管理系统        | 表单、表格、弹窗、下拉、命令面板等组件齐全，配合 Blocks 快速搭建   |
| SaaS 产品                | 品牌定制要求高，多主题 / 暗色模式需求强                            |
| 品牌官网 / 营销页        | 设计稿个性化强，传统组件库风格难以适配，shadcn 可以随意改造        |
| 已在使用 Tailwind 的项目 | 技术栈完全一致，零学习成本，样式体系统一                           |
| 团队自建设计系统         | 以 shadcn 为起点，改造后通过私有 Registry 分发，比从零造轮子快得多 |
| 个人项目 / MVP / 原型    | 安装即用、外观现代，适合快速出效果                                 |
| Next.js / React 全栈项目 | 原生支持 RSC 和 App Router                                         |

### 8.2 需要谨慎评估的场景

| 场景                                   | 顾虑与建议                                                                      |
| -------------------------------------- | ------------------------------------------------------------------------------- |
| 非 React 技术栈                        | 官方仅支持 React；Vue 可看社区移植版 shadcn-vue，Svelte 可看 shadcn-svelte      |
| 未使用 Tailwind 的存量项目             | 需要先引入 Tailwind，注意与现有全局样式的冲突                                   |
| 重度数据密集型企业后台                 | 超复杂表格（虚拟滚动、合并单元格、行内编辑）、复杂表单联动，Ant Design 等更省事 |
| 需要"开箱即用、不想维护组件代码"的团队 | 代码归你也意味着维护归你，上游更新需要手动合并                                  |
| 多项目共享且需严格统一升级             | 需要配套建设私有 Registry 或 monorepo 共享包，否则各项目组件会逐渐分叉          |

### 8.3 选型速查

- 追求**设计自由度 + 品牌定制** → shadcn/ui
- 追求**组件数量 + 开箱即用 + 复杂业务组件** → Ant Design / MUI
- 只需要**无障碍交互能力，样式完全自己写** → 直接用 Radix UI / Base UI / React Aria
- 只是**少量静态页面** → 纯 Tailwind 即可

---

## 九、最佳实践与注意事项

1. **不要直接在页面里大量修改 `components/ui`**：`ui` 目录放基础组件，业务封装放到 `components/` 的其他目录（如 `components/business/`），方便以后对比和合并上游更新。
2. **主题只改变量，不改组件里的颜色**：坚持使用 `bg-primary` 这类语义类，避免在组件中写死 `bg-blue-600`，否则换肤会失效。
3. **自定义变体优先用 `cva`**：不要用三元表达式拼接类名，保持类型安全和可读性。
4. **外部样式一律通过 `cn()` 合并**：自己写的组件也遵循 `className={cn(base, className)}` 约定。
5. **记录改动**：对 shadcn 组件做了结构性修改时，在团队文档或提交信息里说明，方便后续同步上游。
6. **同步上游更新**：可以重新执行 `shadcn add <组件> --overwrite` 到临时分支，再用 Git diff 对比合并，或用 `shadcn diff` 查看差异（视 CLI 版本而定）。
7. **注意 Client 组件边界**：依赖交互的组件（Dialog、Dropdown 等）是客户端组件，在 Next.js 中尽量把它们下沉到叶子节点，避免整页变成 Client Component。
8. **配合 ESLint / Prettier 插件**：使用 `prettier-plugin-tailwindcss` 自动排序类名，保持代码一致性。

---

## 十、总结

- **Tailwind CSS** 用原子化类解决了"样式怎么写、怎么约束、怎么维护"的问题：开发快、无样式污染、体积稳定、刻度统一。v4 进一步把主题配置统一到 CSS 变量中。
- **Tailwind 的自定义能力**（`@theme` 令牌、`@utility` 工具类、`@custom-variant` 变体、`data-*`/`aria-*` 属性样式、CSS 变量参数）让设计规范变成可复用的类名：组件只引用语义、只响应状态，主题只在一处定义。
- **shadcn/ui** 在 Tailwind 之上补齐了"组件怎么写"：用 Radix / Base UI 解决交互与无障碍，用 `cva` 管理变体，用 `cn()` 解决类名冲突，用 CLI 把源码交到你手里。
- 它的三大核心优势：
  - **主题替换**：语义化 CSS 变量驱动，改变量即换肤，暗色模式和多主题天然支持。
  - **高自定义**：`className` 微调 → `cva` 扩展变体 → 直接改源码，三层递进，不再和库"打架"。
  - **完全可控**：代码在自己仓库，不受上游绑架，按需引入、依赖透明，可沉淀为团队自己的组件库。
- 一句话概括：**传统组件库是"租房"，shadcn/ui 是"给你一套精装修的房子，钥匙和图纸都在你手上"。**

---

## 参考资料

- [Tailwind CSS 官方文档](https://tailwindcss.com/docs)
- [Tailwind CSS v4 发布说明](https://tailwindcss.com/blog/tailwindcss-v4)
- [Tailwind CSS 主题变量（@theme）](https://tailwindcss.com/docs/theme)
- [Tailwind CSS 自定义样式（@utility / @custom-variant）](https://tailwindcss.com/docs/adding-custom-styles)
- [shadcn/ui 官方文档](https://ui.shadcn.com/docs)
- [shadcn/ui 主题文档](https://ui.shadcn.com/docs/theming)
- [shadcn/ui Tailwind v4 适配说明](https://ui.shadcn.com/docs/tailwind-v4)
- [Radix UI Primitives](https://www.radix-ui.com/primitives)
- [class-variance-authority](https://cva.style/docs)
- [tailwind-merge](https://github.com/dcastil/tailwind-merge)
