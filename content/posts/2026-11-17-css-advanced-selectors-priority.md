---
title: "2026-11-17 学习日志：结构伪类、表单与伪元素、属性选择器、CSS 三大特性"
date: 2026-11-17T17:00:00+08:00
draft: false
tags: ["CSS", "前端", "结构伪类选择器", "nth-child", "表单伪类", "伪元素", "属性选择器", "CSS三大特性", "优先级", "层叠性", "继承性", "学习日志"]
categories: ["学习笔记"]
summary: "Pink 老师前端课第 37~40 节：先解决「不想给每行都加类名」的痛点——用结构伪类 :first-child / :nth-child(an+b) / :nth-of-type 做表格隔行变色；再用 :checked / :disabled / :focus 这些表单伪类配合 ::before / ::after 伪元素，把原生单选框改造成自定义样式；接着用属性选择器 [type=\"text\"]、[href^=\"http\"] 按「属性特征」而不是类名选元素；最后用 CSS 三大特性（层叠、继承、优先级）把前面所有零散规则收进一套统一的权重计算体系，并总结 11 条今日踩坑。"
---

> 今天的知识地图已经用 Xmind 更新到原来的《HTML 标签 + CSS 样式 · 知识图谱》，文末可以直接下载 `.xmind` 源文件导入 Xmind 继续编辑。

---

## 37 · 结构伪类选择器：不写类名也能选中「第几个」

前面学的基础选择器（类型、类、id）、关系选择器（后代、子代、兄弟），本质上都是"先认出元素是谁，再决定修不修饰它"。但有一类需求很讨厌：

> 表格有 20 行，我要隔行变色；列表有 8 个，我要最后一项不加下边框。

如果按老办法，得写 20 个 `tr` 各给一个 class，改一次数据就崩一次。**结构伪类选择器**就是来解决这件事的：它不看你叫什么名字，只看**你在父元素里排第几**。

### ① `:first-child` 与 `:last-child`

```css
/* 父元素里的第一个子元素，且这个子元素必须是 li */
li:first-child { border-top: none; }

/* 父元素里的最后一个子元素 */
li:last-child { border-bottom: none; }
```

大白话：`:first-child` 是"班里的 1 号"，`:last-child` 是"班里的最后一名"。

⚠️ 最容易踩的坑在这里：`li:first-child` 不是"第一个 li"，而是"**它是父元素的第一个孩子，而且它恰好是 li**"。如果父元素第一个孩子是个 `p`，那这个 `li` 规则谁都不命中。

```html
<ul>
  <p>我是第一个孩子，但不是 li</p>
  <li>A</li>  <!-- 它排第 2，不是 first-child -->
  <li>B</li>
</ul>
```

对照一下：

- `li:first-child` → 上面这段代码里**什么都不选中**；
- `li:first-of-type` → 会选中 `A`（后面讲）。

### ② `:nth-child(n)`：把"第几个"变成公式

```css
/* 第 3 个孩子，并且是 li */
li:nth-child(3) { color: red; }

/* 所有孩子里的偶数位 */
li:nth-child(even) { background: #f5f5f5; }

/* 所有孩子里的奇数位，等价于 2n+1 */
li:nth-child(odd) { background: #fff; }
```

`nth-child()` 括号里可以填三类东西：

| 写法 | 含义 | 等价写法 |
| --- | --- | --- |
| `3` | 第 3 个 | — |
| `odd` | 奇数位（1、3、5…） | `2n+1` |
| `even` | 偶数位（2、4、6…） | `2n` |
| `n` | 全部 | `1n+0` |
| `2n` | 偶数位 | `even` |
| `2n+1` | 奇数位 | `odd` |
| `3n` | 3、6、9、12… | — |
| `3n+1` | 1、4、7、10… | — |
| `n+5` | 从第 5 个开始往后全部 | — |
| `-n+5` | 前 5 个（1~5） | — |

**公式怎么算出来的**：`an+b` 里的 `n` 取 **0、1、2、3……** 逐个代入，把算出来的结果当"序号"去看哪些位置有元素。

以 `2n+1` 为例：

- n=0 → 1
- n=1 → 3
- n=2 → 5
- n=3 → 7

所以是 1、3、5、7……正好是奇数位。

再以 `-n+5` 为例：

- n=0 → 5
- n=1 → 4
- n=2 → 3
- n=3 → 2
- n=4 → 1
- n=5 → 0（没有第 0 个，停）

所以是"前 5 个"。这也是"只给前 N 项加样式"的标准解法：

```css
/* 只给前 5 个标签上色，后面的不上色 */
.tag:nth-child(-n+5) { background: #ffe4e1; }
```

> 大白话：`nth-child` 就是"按座位号点名"。`2n` 是"所有双号"，`-n+5` 是"从第 5 号往前数到 1 号"，也就是前五个。

### ③ `:nth-last-child(n)`：从后往前数

```css
/* 倒数第 1 个（等价 last-child） */
li:nth-last-child(1) { border: none; }

/* 倒数前 3 个：最后三个 */
li:nth-last-child(-n+3) { color: #999; }
```

用在"新消息标记"、"最后 N 条加粗"这种场景很顺手。

### ④ `:nth-child` 家族 vs `:nth-of-type` 家族

这是今天第一个"必须分清"的地方。看代码：

```html
<div class="box">
  <p>我是 p，排第 1</p>
  <span>我是 span，排第 2</span>
  <p>我是 p，排第 3</p>
  <p>我是 p，排第 4</p>
</div>
```

```css
/* 选中的是谁？ */
p:nth-child(2) { color: red; }

/* 选中的又是谁？ */
p:nth-of-type(2) { color: blue; }
```

答案是：

- `p:nth-child(2)` → **一个都不选中**。第 2 个孩子是 `span`，它不是 `p`，所以不满足；
- `p:nth-of-type(2)` → 选中**第 3 个孩子那个 p**。因为 `:nth-of-type` 会先把 `p` 单独挑出来排成一队（第 1 个 p、第 2 个 p、第 3 个 p），再数第 2 个。

一句话区分：

- **`nth-child`：先按「所有兄弟」排位置，再看你类型对不对**（位置错了就出局）；
- **`nth-of-type`：先按「同类型兄弟」排位置，再取第几个**（不同类型不占位）。

同族成员一览：

| 伪类 | 含义 |
| --- | --- |
| `:first-child` | 所有兄弟里的第 1 个，且类型匹配 |
| `:last-child` | 所有兄弟里的最后 1 个，且类型匹配 |
| `:nth-child(n)` | 所有兄弟里的第 n 个，且类型匹配 |
| `:nth-last-child(n)` | 所有兄弟里倒数第 n 个，且类型匹配 |
| `:first-of-type` | 同类型兄弟里的第 1 个 |
| `:last-of-type` | 同类型兄弟里的最后 1 个 |
| `:nth-of-type(n)` | 同类型兄弟里的第 n 个 |
| `:nth-last-of-type(n)` | 同类型兄弟里倒数第 n 个 |
| `:only-child` | 它是父元素**唯一**的孩子（有且仅有它一个） |
| `:only-of-type` | 它是父元素里**该类型唯一**的一个（可以有别的类型） |

`only-child` / `only-of-type` 的实用场景："列表只有一条时，隐藏分隔线"：

```css
/* 只有一条数据时，不要那条多余的横线 */
.item:only-child { border-bottom: none; }
```

### ⑤ 实战：表格隔行变色（斑马纹）

这就是本节标题里"表格隔…"完整的样子。一张成绩表，隔行浅灰，扫读时眼睛不串行。

```html
<table>
  <thead>
    <tr><th>姓名</th><th>语文</th><th>数学</th><th>英语</th></tr>
  </thead>
  <tbody>
    <tr><td>张三</td><td>92</td><td>88</td><td>95</td></tr>
    <tr><td>李四</td><td>85</td><td>91</td><td>79</td></tr>
    <tr><td>王五</td><td>78</td><td>95</td><td>88</td></tr>
    <tr><td>赵六</td><td>90</td><td>82</td><td>86</td></tr>
  </tbody>
</table>
```

```css
/* 关键：限定在 tbody 里数，表头不参与 */
tbody tr:nth-child(even) {
  background: #f7f7f7;
}

/* 鼠标悬停高亮整行，比斑马纹更好用 */
tbody tr:hover {
  background: #e8f4ff;
}

/* 表头单独处理，别被斑马纹影响 */
thead th {
  background: #2c3e50;
  color: #fff;
}
```

**为什么一定要写 `tbody`？** 因为 `tr:nth-child()` 是相对它自己的父元素计数的。`thead` 里的 `tr` 和 `tbody` 里的 `tr` 父元素不同，各数各的。如果你偷懒写成 `tr:nth-child(even)`，表头所在的 `tr` 也会被当成"第 1 个孩子"，整体错位一格；而且有些表格干脆不写 `thead`/`tbody`（浏览器会自动补 `tbody`），表头和数据行混在同一个 tbody 里，第 1 行就是表头：

```css
/* 表头和数据行混排时：奇数行才是数据行奇数位 */
tr:nth-child(odd) { background: #f7f7f7; }
```

所以更稳的写法是：**给表头单独一个 `thead`，斑马纹只作用 `tbody tr`**；实在混排就用 `odd` 并注释说明。

三个"隔行"以外的常用变体：

```css
/* 前 3 名加奖牌色 */
tbody tr:nth-child(-n+3) { background: #fff8dc; }

/* 最后一列（每行最后一个 td）右对齐 */
td:last-child { text-align: right; }

/* 每 4 行加一条更明显的分隔线 */
tbody tr:nth-child(4n) { border-bottom: 2px solid #ddd; }
```

---

## 38 · 表单选择器（表单伪类）与伪元素选择器

### ① 表单相关的伪类：按「状态」选元素

普通选择器选的是"你是谁"（标签、类、id、属性）；**状态伪类**选的是"你现在处于什么状态"。表单控件天生有状态，所以这一组伪类几乎都围绕 `input` 打转。

| 伪类 | 命中条件 | 典型用途 |
| --- | --- | --- |
| `:checked` | 单选/复选/下拉项处于**选中**状态 | 自定义单选框、复选框 |
| `:disabled` | 控件被禁用（`disabled`） | 禁用按钮灰化、加禁止鼠标 |
| `:enabled` | 控件可用（未禁用） | 可用输入框加边框 |
| `:focus` | 控件**获得焦点**（正在输入） | 输入框高亮、聚焦动画 |
| `:required` / `:optional` | 必填 / 选填 | 给必填项加星号 |
| `:read-only` / `:read-write` | 只读 / 可读写 | 只读框灰底提示 |
| `:placeholder-shown` | 占位符正显示（即用户还没输入） | 浮动标签（Floating Label） |

前四个是这节课的核心，后几个是同一类思路的扩展，知道有就行。

**例 1：输入框聚焦高亮 + 禁止禁用控件**

```css
/* 默认态：细边框 */
input[type="text"] {
  border: 1px solid #ccc;
  padding: 8px 12px;
  outline: none;              /* 去掉浏览器默认的蓝框 */
  transition: border-color .2s, box-shadow .2s;
}

/* 聚焦态：蓝色边框 + 外发光 */
input[type="text"]:focus {
  border-color: #1e90ff;
  box-shadow: 0 0 0 3px rgba(30, 144, 255, .15);
}

/* 禁用态：灰底 + 禁止光标 */
input:disabled {
  background: #f0f0f0;
  color: #999;
  cursor: not-allowed;
}

/* 可用态（和 disabled 相反，常用于"覆盖"默认样式） */
input:enabled {
  background: #fff;
}
```

> 注意 `outline: none` 单独用是有无障碍风险的：键盘操作时用户看不到焦点在哪。所以正确做法是 **`outline: none` 的同时自己给一个可见的 `box-shadow` / `border-color`**，别只是把它删掉。

**例 2：自定义单选框（把系统小圆点换成自己的样式）**

系统原生 `radio` 长相各平台不一致，也不好看。标准做法是"藏起真的，画个假的"：

```html
<label class="radio">
  <input type="radio" name="gender" value="male">
  <span class="dot"></span>
  男
</label>
<label class="radio">
  <input type="radio" name="gender" value="female">
  <span class="dot"></span>
  女
</label>
```

```css
/* 1. 真的 input 藏起来，但不能用 display:none（会失去键盘可达性） */
.radio input {
  position: absolute;
  opacity: 0;
  width: 0;
  height: 0;
}

/* 2. 假的圆点 */
.radio .dot {
  display: inline-block;
  width: 16px;
  height: 16px;
  border: 1px solid #bbb;
  border-radius: 50%;
  margin-right: 6px;
  vertical-align: middle;
  position: relative;
}

/* 3. 选中时：假圆点填色 + 中间小白点 */
.radio input:checked + .dot {
  border-color: #1e90ff;
  background: #1e90ff;
}

.radio input:checked + .dot::after {
  content: "";
  position: absolute;
  left: 50%;
  top: 50%;
  width: 6px;
  height: 6px;
  background: #fff;
  border-radius: 50%;
  transform: translate(-50%, -50%);
}
```

这段代码把前面几节课的东西全用上了：`:checked`（状态伪类）、`+`（相邻兄弟）、`::after`（伪元素）、`transform` 居中。**这是"知识点打通"的典型例子**——单独背伪类没用，能拼起来才算会。

复选框同理，`border-radius` 改成 3px，选中时用 `::after` 画一个"√"：

```css
.checkbox input:checked + .box::after {
  content: "✓";
  color: #fff;
  font-size: 12px;
  line-height: 16px;
  text-align: center;
}
```

### ② 伪元素选择器：往元素里"插入"不存在的内容

**伪元素（pseudo-element）** 不是页面里真实存在的标签，而是 CSS 凭空造出来的"虚拟子元素"。它有两个最重要的成员：`::before` 和 `::after`。

语法要点（三条，缺一不可）：

1. 必须写 **`content`** 属性，哪怕内容是空字符串 `content: "";`，否则整个伪元素不生成；
2. 默认是 `display: inline`，要设宽高就必须转成 `inline-block` / `block`；
3. 现代写法用**双冒号** `::before`（CSS3 规范区分伪类和伪元素），老写法 `:before` 也能用，但建议统一双冒号。

```css
/* 最基础：在一段文字前后加引号 */
.quote::before { content: "「"; color: #1e90ff; }
.quote::after  { content: "」"; color: #1e90ff; }
```

```css
/* 列表项前面的小方块（替代 list-style，可控性更强） */
.item::before {
  content: "";
  display: inline-block;      /* 转成行内块才能设宽高 */
  width: 6px;
  height: 6px;
  background: #1e90ff;
  border-radius: 50%;
  margin-right: 8px;
  vertical-align: middle;
}
```

**经典用途 1：清除浮动（clearfix）**

```css
.clearfix::after {
  content: "";
  display: block;      /* 必须块级，才能撑开父容器 */
  clear: both;
  height: 0;
  visibility: hidden;
}
```

> 为什么用 `::after` 而不用 `overflow: hidden`？因为 `overflow: hidden` 会裁掉溢出的内容（比如阴影、下拉菜单），而 clearfix 只影响高度计算，副作用最小。

**经典用途 2：字体图标 / 装饰角标**

```css
/* 在按钮右侧加一个"下一张"箭头，不用额外写标签 */
.next-btn::after {
  content: "→";
  margin-left: 6px;
  transition: margin-left .2s;
}
.next-btn:hover::after { margin-left: 12px; }   /* hover 时箭头右移，做微动效 */
```

**经典用途 3：图片 hover 遮罩层**

```css
.cover {
  position: relative;
  overflow: hidden;
}
.cover::after {
  content: "查看详情";
  position: absolute;
  left: 0; top: 0;
  width: 100%; height: 100%;
  background: rgba(0, 0, 0, .6);
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity .3s;
}
.cover:hover::after { opacity: 1; }
```

### ③ 四个"文字级"伪元素

除了 `::before` / `::after`，还有四个作用于文字某一部分的伪元素（了解即可，用得少但很出彩）：

| 伪元素 | 作用 | 典型效果 |
| --- | --- | --- |
| `::first-letter` | 第一个字母/汉字 | 杂志式首字下沉 |
| `::first-line` | 第一行文字 | 首行特殊排版（随宽度变化） |
| `::selection` | 用户**鼠标选中**的文字 | 自定义选中高亮色 |
| `::placeholder` | 输入框占位文字 | 占位符颜色、字号 |

```css
/* 首字下沉：小说、文章开头 */
.article p::first-letter {
  font-size: 2.2em;
  float: left;
  line-height: 1;
  margin-right: 6px;
  color: #1e90ff;
}

/* 选中文字的品牌色 */
::selection {
  background: #1e90ff;
  color: #fff;
}

/* 占位符统一灰色小字 */
input::placeholder {
  color: #aaa;
  font-size: 13px;
}
```

> 踩坑：`::first-letter` 和 `::first-line` **只能用在块级容器**（p、div、li 等），用在 `span` 这种行内元素上无效。

### ④ 一个容易混的点：伪类 vs 伪元素

| | 伪类 | 伪元素 |
| --- | --- | --- |
| 冒号 | 单冒号 `:` | 双冒号 `::`（老写法单个冒号） |
| 选的是什么 | 元素在**某种状态**下的样子 | 元素内部**虚拟出来的一部分** |
| 例子 | `:hover` `:checked` `:nth-child` `:focus` | `::before` `::after` `::first-letter` `::selection` |
| 权重 | 等同于**类**（0,0,1,0） | 等同于**标签**（0,0,0,1） |

最后一行非常关键：`.item::before` 的总权重是 `0,0,1,0 + 0,0,0,1 = 0,0,1,1`。这一点在第 40 节的优先级计算里会反复用到。

---

## 39 · 属性选择器：按"身上带的特征"选元素

前面所有的选择器，要么靠名字（`p`、`.box`、`#header`），要么靠位置（`:nth-child`），要么靠状态（`:checked`）。**属性选择器**提供第四种思路：**看你身上有没有某个属性、属性值长什么样**。

语法统一是 `[属性名 匹配符 值]`，中括号包起来。

### ① 七种匹配方式（按精确度从高到低）

| 写法 | 含义 | 例子 | 命中 |
| --- | --- | --- | --- |
| `[attr]` | 只要有这个属性就选 | `[disabled]` | 所有带 disabled 的控件 |
| `[attr="v"]` | 值**完全等于** v | `[type="text"]` | `type="text"` |
| `[attr~="v"]` | 值里有一个**独立单词**等于 v（空格分隔） | `[class~="btn"]` | `class="btn primary"` |
| `[attr\|="v"]` | 值等于 v，或以 **`v-` 开头** | `[lang\|="zh"]` | `lang="zh"`、`lang="zh-CN"` |
| `[attr^="v"]` | 值**以 v 开头** | `[href^="https"]` | `https://...` |
| `[attr$="v"]` | 值**以 v 结尾** | `[src$=".png"]` | `a.png`、`b.png` |
| `[attr*="v"]` | 值**包含 v 子串**（任意位置） | `[title*="教程"]` | "Vue 教程"、"教程合集" |

```css
/* ① 存在即选中：所有被禁用的 input */
input[disabled] { opacity: .6; }

/* ② 精确匹配：只选文本框，不动按钮和复选框 */
input[type="text"] { border: 1px solid #ddd; }

/* ③ 包含单词：class 里有 btn 就算（多类名场景） */
[class~="btn"] { padding: 8px 16px; }

/* ④ 前缀带连字符：语言相关 */
[lang|="zh"] { font-family: "Microsoft YaHei", sans-serif; }

/* ⑤ 开头匹配：外链加小箭头标记 */
a[href^="http"]::after { content: " ↗"; font-size: 12px; }

/* ⑥ 结尾匹配：只给 png 图片加边框 */
img[src$=".png"] { border: 1px solid #eee; }

/* ⑦ 子串匹配：标题里带"教程"的都标黄 */
[title*="教程"] { background: #fffbe6; }
```

### ② 几个必须记住的细节

1. **`~=` 和 `*=` 长得像，语义完全不同**：
   - `[class~="btn"]` 只有 `class="btn"`、`class="btn primary"` 才命中，`class="btn-primary"` **不命中**（因为它是一个词 `btn-primary`，不是 `btn`）；
   - `[class*="btn"]` 则连 `class="btn-primary"` 也命中，因为字符串里包含 `btn`。
   - 一句话：`~=` 比的是"词"，`*=` 比的是"字符串"。

2. **`|=` 只认连字符 `-`**，不是随便什么前缀。`[lang|="zh"]` 命中 `zh`、`zh-CN`，但**不命中** `zh_CN`。

3. **`^=` `$=` `*=` 比较的是完整字符串**，包括大小写（CSS 属性值匹配默认大小写敏感，除非属性本身不敏感，比如 `type`）。

4. **属性选择器可以叠加**，也可以和类型、类、伪类组合：

```css
/* 所有以 disabled 标记的 input 按钮 */
input[type="submit"][disabled] { cursor: not-allowed; }

/* 所有 a 标签中，href 以 http 开头的（外链） */
a[href^="http"] { color: #e74c3c; }

/* 文件类型图标：不同后缀不同前缀 */
a[href$=".pdf"]::before { content: "📄 "; }
a[href$=".zip"]::before { content: "🗜 "; }
```

5. **权重是"类级别"**：`[type="text"]` 的权重是 `0,0,1,0`，和 `.text-input` 一样重。所以出现"属性选择器和类选择器打架"时，谁后面写谁赢——这是第 40 节要讲的内容。

### ③ 实战：用属性选择器统一表单样式

```html
<form class="login">
  <input type="text" placeholder="用户名" required>
  <input type="password" placeholder="密码" required>
  <input type="email" placeholder="邮箱">
  <input type="submit" value="登录">
  <input type="submit" value="注册" disabled>
</form>
```

```css
/* 一条规则统一所有文本类输入框，不用给每个都加 class */
.login input:not([type="submit"]) {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-bottom: 12px;
}

/* 用属性选择器区分两种按钮，不用写两个类 */
.login input[type="submit"]:enabled  { background: #1e90ff; color: #fff; }
.login input[type="submit"]:disabled { background: #ccc;  color: #fff; }

/* 必填项在占位符后面加星号（用相邻兄弟 + 伪元素做不到，用 required 判断） */
.login input[required] { border-left: 3px solid #e74c3c; }
```

顺带一个反向选择器 `:not()`（今天顺手补的，很常用）：

```css
/* 除 submit 之外的所有 input */
input:not([type="submit"]) { /* ... */ }

/* 不是最后一行的所有行 */
tr:not(:last-child) { border-bottom: 1px solid #eee; }
```

> `:not()` 括号里可以放类型、类、属性、伪类，**不能放伪元素**，也**不能嵌套**（CSS3 限制，CSS4 才允许嵌套）。权重取括号里最高的那个。

---

## 40 · CSS 三大特性：层叠性、继承性、优先级

到这里，选择器已经学了一整套：基础、关系、结构伪类、状态伪类、属性。**但知道怎么选元素，不等于知道"选完之后谁说了算"**。同一个 `p` 可能被 5 条规则同时命中，颜色到底听谁的？这就是三大特性要解决的问题。

### ① 层叠性（Cascading）

定义：**多个选择器命中同一个元素的同一个属性时，冲突的按规则取一个，不冲突的全部生效。**

```css
p { color: red; font-size: 14px; }
p { color: blue; }         /* 只覆盖 color，font-size 保留 */
```

结果：`color: blue`（后写的赢），`font-size: 14px`（没冲突，保留）。

这就是"层叠"——像千层饼一样一层层叠上去，**没被覆盖的层都留着**。

层叠的两个前提：

1. **权重相同** → 后写的覆盖先写的（"后来者居上"）；
2. **权重不同** → 权重大的赢，跟写的先后无关。

```css
/* 权重 0,0,0,1 */
p { color: red; }

/* 权重 0,0,1,0 —— 就算写在前面也赢 */
.text { color: blue; }
```

> 大白话：层叠性解决的是"平级打架"，优先级解决的是"上级压下级"。

### ② 继承性（Inheritance）

定义：**子元素会自动拥有父元素身上"可以继承"的样式，不需要额外写规则。**

```css
body {
  font-family: "Microsoft YaHei", sans-serif;
  font-size: 16px;
  color: #333;
  line-height: 1.6;
}
```

写这一条，全站文字都跟着变——`body` 里所有后代（p、li、a、span……）全部继承。这就是博客里"统一字体"最简单的写法。

**哪些能继承？**

| 类别 | 具体属性 |
| --- | --- |
| 字体类 | `font-family` `font-size` `font-style` `font-weight` `font` |
| 文本类 | `color` `line-height` `text-align` `text-indent` `letter-spacing` `word-spacing` `text-transform` `white-space` |
| 列表 | `list-style` `list-style-type` `list-style-position` |
| 其他 | `visibility` `cursor` `quotes` `direction` |

**哪些不能继承？**

| 类别 | 具体属性 | 原因 |
| --- | --- | --- |
| 盒模型 | `width` `height` `margin` `padding` `border` | 每个盒子尺寸应各自独立 |
| 背景 | `background` `background-color` | 背景不该蔓延 |
| 布局 | `display` `position` `float` `clear` `overflow` `z-index` | 位置关系必须显式声明 |
| 其他 | `text-decoration`（不继承给子元素，但视觉上会"穿透"显示） | 特殊 |

**两个必须记住的"继承陷阱"**：

1. **`<a>` 标签会破坏继承**。浏览器给 `a` 设了默认的 `color: -webkit-link`（蓝色），而"元素自身的默认声明"优先于"从父级继承来的值"。所以你在 `body` 里写 `color: #333`，`a` 依然是蓝的。解决办法：

```css
a {
  color: inherit;            /* 显式声明：继承父级的颜色 */
  text-decoration: none;
}
```

2. **继承来的样式权重是 0**。这意味着它对任何"直接写在元素上的声明"都没有抵抗力，连通配符都能压过它：

```css
body { color: red; }        /* 子元素继承到红 */
* { color: green; }         /* 通配符直接命中所有元素，优先级高于"继承" */
```

结果全绿。这也是为什么"`* { margin: 0; padding: 0; }`"能清掉所有间距——通配符虽然权重是 0，但它是"直接命中"，比继承强。

### ③ 优先级（Specificity）

这是三大特性里最重要、也最容易考的一条。CSS 用**四位权重**来比较：

```
(行内样式, ID 选择器, 类/属性/伪类, 元素/伪元素)
```

| 来源 | 权重 | 记忆 |
| --- | --- | --- |
| `!important` | 最高（∞） | 强拆一切，慎用 |
| 行内 `style="..."` | `(1,0,0,0)` | 写在标签上，最"近" |
| ID 选择器 `#id` | `(0,1,0,0)` | 一人一号，尽量少用 |
| 类 `.cls`、属性 `[attr]`、伪类 `:hover` | `(0,0,1,0)` | 日常主力 |
| 元素 `p`、伪元素 `::before` | `(0,0,0,1)` | 最轻 |
| 通配符 `*`、继承 | `(0,0,0,0)` | 垫底 |

**比较规则：从左往右逐位比，第一位大就赢，不看后面；而且是"位"不是"数"，不进位。**

```css
/* A: (0,0,1,0) 一个类 */
.text { color: red; }

/* B: (0,0,0,2) 两个标签 */
div p { color: blue; }
```

A 赢。虽然 B 有两个标签，但第一位 `0,0,1` 已经大于 `0,0,0`，**1 个类和 11 个标签相比，还是类赢**（不存在"11 个标签凑成 1 个类"的进位规则）。

**复合选择器的权重 = 各部分相加**：

| 选择器 | 计算 | 权重 |
| --- | --- | --- |
| `p` | 1 个元素 | `0,0,0,1` |
| `.box p` | 1 类 + 1 元素 | `0,0,1,1` |
| `.box > p` | 关系符 `>` 不算权重 | `0,0,1,1` |
| `ul li a` | 3 元素 | `0,0,0,3` |
| `#header .nav li` | 1 id + 1 类 + 1 元素 | `0,1,1,1` |
| `a:hover` | 1 元素 + 1 伪类 | `0,0,1,1` |
| `.item::before` | 1 类 + 1 伪元素 | `0,0,1,1` |
| `input[type="text"]:focus` | 1 属性 + 1 伪类 | `0,0,2,0` |
| `*` | 通配符不加权 | `0,0,0,0` |

> 注意最后一行：**关系选择器（空格、`>`、`+`、`~`）本身不加权重**，`div p` 是 `0,0,0,2` 完全是因为有两个标签，不是因为有个空格。

**分组选择器各算各的**：

```css
/* 逗号不是"共享权重"，每一项单独计算 */
h1, .title, #main { color: #333; }
```

上面这条里，`h1` 是 `0,0,0,1`，`.title` 是 `0,0,1,0`，`#main` 是 `0,1,0,0`。谁被哪个命中，就按哪个的权重去比。

**`!important` 的位置**：

```css
p { color: red !important; }        /* 最高，压过一切普通声明 */
#main p { color: blue; }            /* 权重再高也压不过 !important */
```

`!important` 会打破整个权重体系，只在两种场景用：

1. 覆盖第三方库/框架的样式（改不动源码时）；
2. 临时调试。

**别用它给自己埋坑**：一旦用多了，后面想改就得继续加 `!important`，层层叠上去就再也理不清了。

### ④ 三大特性串起来看：一个真实冲突现场

```html
<div id="app" class="wrap">
  <ul class="list">
    <li class="item">内容</li>
  </ul>
</div>
```

```css
/* ① 权重 0,0,0,1 —— 元素 */
li { color: #333; }

/* ② 权重 0,0,1,1 —— 类 + 元素 */
.list li { color: #666; }

/* ③ 权重 0,1,1,1 —— id + 类 + 元素 */
#app .list li { color: #999; }

/* ④ 权重 0,0,2,1 —— 两个类 + 元素，还是打不过 ③ */
.wrap .list li { color: #000; }

/* ⑤ 行内样式 1,0,0,0 —— 直接碾压前面所有 */
/* <li style="color: red">内容</li> */
```

最终颜色：如果没写行内样式，是 `#999`（③ 赢）；写了行内，就是 `red`。

**优先级相同怎么办？** 看书写顺序，后写的赢：

```css
.list li { color: #666; }
.list li { color: #333; }   /* 权重一样，后写覆盖 */
```

### ⑤ 一条"少踩坑"的优先级实践原则

1. **优先用类选择器**，权重 `0,0,1,0` 够用又好改；
2. **尽量不用 ID 做样式选择器**（`#id` 权重太高，后面想覆盖只能加 `!important` 或嵌套更多层）；
3. **选择器嵌套别超过 3 层**，`#app .wrap .list li a` 这种权重爆炸且难维护；
4. **`!important` 只做兜底**，不做常规手段；
5. **写 CSS 前先想清楚层级**：同一个元素身上不要出现两条"权重相同、属性冲突"的规则，否则以后改起来就是猜。

---

## 今日踩坑小结

1. **`:first-child` 不是"第一个 li"，而是"父元素第一个孩子，且类型匹配"**——父元素第一个孩子不是 li 时，这条规则直接失效；
2. **`:nth-child(n)` 的位置是"所有兄弟一起排"，`:nth-of-type(n)` 是"同类型单独排"**，两者在混合元素容器里结果完全不同；
3. **`an+b` 里的 n 从 0 开始代入**，`-n+5` 是"前 5 个"，`2n+1` 是奇数位，别把 `2n` 当成"前两个"；
4. **表格斑马纹一定要限定 `tbody tr`**，否则表头 `thead` 的 `tr` 会参与计数导致整体错位；表头和数据混在一个 `tbody` 时，要用 `odd` 而不是 `even`；
5. **`:checked` 必须配合 `+` / `~` 兄弟选择器才能改到"假控件"**，因为被选中的 input 和被画的 span 是兄弟关系，不是父子；
6. **自定义单选框时不要用 `display: none` 藏 input**，会失去键盘可达性，用 `opacity: 0` + `position: absolute`；
7. **`::before` / `::after` 的 `content` 忘了写 → 整个伪元素不生成**，这是最高频的"写了没反应"；
8. **伪元素默认 `display: inline`**，要设宽高必须转 `inline-block` / `block`，否则 `width`/`height` 完全不生效；
9. **`::first-letter` / `::first-line` 只能用在块级容器**，写在 `span` 上无效；
10. **`[attr~="v"]` 比的是"词"，`[attr*="v"]` 比的是"字符串"**，`class="btn-primary"` 能被 `*=` 命中，但不会被 `~=` 命中；
11. **属性选择器的权重等同类**（`0,0,1,0`），不是标签级别，和类选择器打架时看书写顺序；
12. **继承的样式权重是 0**，任何直接声明（哪怕通配符）都能压过它；`<a>` 因为自带默认 color 会"破坏继承"，需要 `color: inherit`；
13. **权重按位比较、不进位**，11 个类也换不来 1 个 id；
14. **关系选择器不参与权重计算**，`div p` 是 `0,0,0,2` 是因为两个标签，不是因为那个空格；
15. **`:not()` 里不能放伪元素，CSS3 不支持嵌套**，权重取括号内最高项。

---

## 综合练习：把今天的知识点串成一个登录页

需求：

- 表单输入框统一圆角、聚焦高亮；
- 密码框用属性选择器区分，不加 class；
- 提交按钮禁用时灰化、可用时蓝色；
- 一个"同意协议"自定义复选框；
- 表格成绩隔行变色。

```html
<div class="login-box">
  <h2>登录</h2>
  <form>
    <input type="text" placeholder="用户名" required>
    <input type="password" placeholder="密码" required>
    <label class="agree">
      <input type="checkbox" checked>
      <span class="mark"></span>
      我已阅读并同意协议
    </label>
    <input type="submit" value="登录">
    <input type="submit" value="注册" disabled>
  </form>
</div>

<table class="score">
  <thead><tr><th>姓名</th><th>成绩</th></tr></thead>
  <tbody>
    <tr><td>张三</td><td>92</td></tr>
    <tr><td>李四</td><td>85</td></tr>
    <tr><td>王五</td><td>78</td></tr>
  </tbody>
</table>
```

```css
/* 1. 属性选择器统一输入框：排除 submit */
.login-box input:not([type="submit"]):not([type="checkbox"]) {
  width: 100%;
  padding: 10px 12px;
  margin-bottom: 12px;
  border: 1px solid #ddd;
  border-radius: 6px;
  outline: none;
}

/* 2. 聚焦状态伪类 */
.login-box input:focus {
  border-color: #1e90ff;
  box-shadow: 0 0 0 3px rgba(30, 144, 255, .15);
}

/* 3. 必填提示：属性选择器 */
.login-box input[required] {
  border-left: 3px solid #e74c3c;
}

/* 4. 两种按钮靠 enabled / disabled 区分 */
.login-box input[type="submit"] {
  width: 100%;
  padding: 10px;
  margin-top: 6px;
  border: none;
  border-radius: 6px;
  color: #fff;
  cursor: pointer;
}
.login-box input[type="submit"]:enabled  { background: #1e90ff; }
.login-box input[type="submit"]:disabled {
  background: #ccc;
  cursor: not-allowed;
}

/* 5. 自定义复选框：:checked + 兄弟 + 伪元素 */
.agree input { position: absolute; opacity: 0; width: 0; height: 0; }
.agree .mark {
  display: inline-block;
  width: 16px; height: 16px;
  border: 1px solid #bbb;
  border-radius: 3px;
  margin-right: 6px;
  vertical-align: middle;
  position: relative;
}
.agree input:checked + .mark {
  border-color: #1e90ff;
  background: #1e90ff;
}
.agree input:checked + .mark::after {
  content: "✓";
  position: absolute;
  left: 2px; top: -1px;
  color: #fff;
  font-size: 12px;
}

/* 6. 斑马纹：只数 tbody */
.score tbody tr:nth-child(even) { background: #f7f7f7; }
.score tbody tr:hover { background: #e8f4ff; }
.score thead th { background: #2c3e50; color: #fff; }
```

一道题里用到了：属性选择器、`:not()`、`:focus`、`:required`、`:enabled`、`:disabled`、`:checked`、相邻兄弟 `+`、伪元素 `::after`、结构伪类 `:nth-child`、层叠与优先级。**这就是"知识点连通"的样子。**

---

## 自测清单

- [ ] 说出 `:first-child` 和 `:first-of-type` 的区别，并各举一个"会失效"的场景
- [ ] 写出"选中前 5 个元素"的 `nth-child` 表达式，并解释 n 取值过程
- [ ] 表格斑马纹为什么必须写 `tbody tr`？不写会发生什么？
- [ ] 把原生单选框改造成自定义样式，最少需要几个组件？分别承担什么角色？
- [ ] `::before` 写了却没反应，列出三个可能原因
- [ ] `[class~="btn"]` 和 `[class*="btn"]` 对 `class="btn-primary"` 分别命中吗？
- [ ] `[lang|="zh"]` 能匹配 `lang="zh_CN"` 吗？
- [ ] 默写四位权重表，并算出 `input[type="text"]:focus` 的权重
- [ ] `div p` 和 `.box p` 的权重分别是多少？关系符要不要计数？
- [ ] 为什么说"11 个类换不来 1 个 id"？
- [ ] 继承来的样式权重是多少？为什么 `* { color: green }` 能盖掉 `body { color: red }`？
- [ ] `<a>` 为什么不继承父级 `color`，怎么修？
- [ ] 层叠性和优先级分别在解决什么问题？各举一个例子
- [ ] `!important` 什么时候可以用，什么时候是坑？

---

## 附：今天的思维导图

- Xmind 源文件：[下载 2026-11-17-css-advanced-selectors.xmind](/files/2026-11-17-css-advanced-selectors.xmind)

导入方式：Xmind 打开 → 文件 → 导入 → 选择该 `.xmind` 文件。这份图谱包含从 HTML 标签体系到今天的结构伪类、属性选择器、三大特性的完整脉络，可以按节点展开复习。

> 下一篇该进 CSS 盒子模型了：`border` / `padding` / `margin` 三兄弟一旦拆开，页面布局才真正开始有"尺寸感"。今天学的优先级，会在盒模型叠加的时候天天用到 💪
