---
title: "2026-11-16 学习日志：CSS 关系选择器、字体样式与文本排版全解析"
date: 2026-11-16T21:10:00+08:00
draft: false
tags: ["CSS", "前端", "关系选择器", "后代选择器", "子代选择器", "兄弟选择器", "字体样式", "文本排版", "伪类选择器", "学习日志"]
categories: ["学习笔记"]
summary: "Pink 老师前端课第 28~36 节：从单个选择器进阶到「关系选择器」——后代、子代、相邻兄弟、通用兄弟，靠空格和三个符号就能精准定位元素；接着用《登高》做综合案例，正式进入字体样式（color / font-family / font-size / font-style / font-weight / font 简写）与文本排版（对齐、缩进、装饰线、行高、字间距）；最后补上分组选择器和链接伪类 LVHA、用户行为伪类 hover / focus，并用 Xmind 把这一天的知识串成一张网。"
---

![CSS 关系选择器与字体文本排版思维导图](/images/css-relations-font-text-mindmap.png)

> 今天的知识地图已经用 Xmind 整理好了，文末可以直接下载 `.xmind` 源文件导入 Xmind 继续编辑。

---

## 28 · 关系选择器：后代选择器与子代选择器

前面学的类型、类、id 选择器都是「直接点名某个元素」。但真实页面是有层级结构的：`div` 里面有 `ul`，`ul` 里面有 `li`，`li` 里面还有 `a`。这时候光靠 `.box` 这种点名方式就不够精准了——我想要的是「`.box` 里面的 `li`」，而不是全页面所有的 `li`。

这就是**关系选择器（也叫复合选择器、层级选择器）**要解决的问题：用两个及以上选择器，按**层级关系**组合起来定位元素。

### ① 后代选择器（空格）

语法：`选择器1 选择器2`，中间用**空格**隔开。

```css
/* 选中 .box 里面所有的 p —— 儿子、孙子、重孙子，全都要 */
.box p {
  color: #333;
  line-height: 1.8;
}
```

```html
<div class="box">
  <p>我是儿子，被选中</p>
  <div class="inner">
    <p>我是孙子，也被选中</p>
    <ul>
      <li><p>我是重孙子，照样选中</p></li>
    </ul>
  </div>
</div>
<p>我是外面的 p，安全，没被选中</p>
```

几个要点：

1. **只认「后代」，不看层级深浅**——只要在 `.box` 内部，隔多少层都能被选中；
2. **空格绝对不能省**：写成 `.boxp` 就成了一个叫 `boxp` 的类，跟层级完全没关系了；
3. 两边的选择器可以是任意基础选择器，甚至可以继续套：

```css
/* ul 下所有 li 里的 a */
ul li a {
  text-decoration: none;
  color: #1e90ff;
}

/* .nav 下、类名为 .item 的元素里的 span */
.nav .item span {
  font-size: 12px;
}

/* 三层也可以，但不建议太深，超过三层就该反思结构了 */
.header .nav .menu li {
  margin-right: 20px;
}
```

> 大白话：后代选择器是「查户口」——只要你的户口落在我家，不管住几楼几号，都算我的人。

### ② 子代选择器（`>`）

语法：`选择器1 > 选择器2`，用大于号 `>` 连接。

它比后代选择器「挑剔」：**只选直接子元素（亲儿子），孙子不算**。

```css
/* 只选 .box 的直接子元素 p */
.box > p {
  font-weight: bold;
}
```

```html
<div class="box">
  <p>亲儿子 —— 选中 ✅</p>
  <div class="inner">
    <p>孙子 —— 不选 ❌</p>
  </div>
</div>
```

对比记忆：

| 写法 | 含义 | 选的范围 |
| --- | --- | --- |
| `.box p` | 后代（空格） | 所有后代，含子孙 |
| `.box > p` | 子代（`>`） | 只选直接子元素 |

什么时候用 `>`？**当你只想管一层的时候**。比如侧边栏只想调它自己里面的标题，不想影响嵌套卡片的标题，用 `>` 就很干净。

### ③ 关于权重的澄清

有个常见误解：「写了层级选择器，权重就变高了」。

其实关系选择器**本身不加权重**，它的权重就是把各部分的选择器权重**加起来**：

```css
/* 权重 0,0,1,1 —— 一个类 + 一个标签 */
.box p { }

/* 权重 0,0,1,2 —— 一个类 + 两个标签 */
.box div p { }
```

所以 `div p` 比 `p` 权重高，是因为多了一个 `div` 标签，而不是因为中间那个空格。

---

## 29 · 关系选择器：兄弟选择器

兄弟关系指的是**同一父元素下、平级的元素**。兄弟选择器有两种：

### ① 相邻兄弟选择器（`+`）

语法：`A + B`，选中 **A 后面紧挨着的那个** 同级兄弟 B。

```css
/* 选中 h2 后面紧挨着的那一个 p */
h2 + p {
  color: #888;
  font-style: italic;
}
```

```html
<h2>小标题</h2>
<p>我是紧挨着的 p，被选中 ✅</p>
<p>我隔了一个，不选 ❌</p>
```

### ② 通用兄弟选择器（`~`）

语法：`A ~ B`，选中 **A 后面所有** 同级兄弟 B（前面的不算）。

```css
/* 选中 h2 后面所有的 p */
h2 ~ p {
  text-indent: 2em;
}
```

```html
<h2>小标题</h2>
<p>选中 ✅</p>
<p>选中 ✅</p>
<div>我是 div，类型不符，不选</div>
<p>我也选中 ✅</p>
```

### ③ 一条铁律 + 一个实用场景

**铁律：兄弟选择器只能往后选，永远选不到前面的兄弟。**

CSS 是「向下、向后」流动的语言，没有「前一个兄弟」这种选择器（真正的上一个兄弟要靠 `:has()` 之类的现代特性绕，不在今天范围内）。

兄弟选择器最实用的场景是**「列表项之间的分割线」**：

```css
/* 每一项都加上边框，除了第一项 —— 完美避免首尾多余线条 */
.list li + li {
  border-top: 1px solid #eee;
}
```

如果用普通写法，就得写 `border-top` 然后单独把 `:first-child` 的边框去掉，多两行代码。用 `li + li` 一行搞定，这是兄弟选择器最经典的用法。

---

## 30 · 综合案例：登高千古第一绝句

这一节是把前面学的选择器 + 马上要学的文本属性串起来做一个小页面——给杜甫的《登高》排版。

目标效果：标题居中、诗句正文缩进且舒展、作者署名右对齐、整体有一点古风的留白感。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>登高 · 杜甫</title>
  <style>
    /* 1. 页面底色和字体 */
    body {
      background: #f5f1e8;
      font-family: "KaiTi", "STKaiti", serif;
      color: #3b2f23;
    }

    /* 2. 用后代选择器把样式锁在 .poem 内部，不影响页面其他地方 */
    .poem {
      width: 520px;
      margin: 80px auto;
      padding: 40px 50px;
      background: #fffdf8;
      border: 1px solid #e5ddcc;
      border-radius: 4px;
    }

    /* 3. 标题：居中 + 字间距拉开 */
    .poem .title {
      text-align: center;
      font-size: 26px;
      letter-spacing: 4px;
      font-weight: bold;
      margin-bottom: 8px;
    }

    /* 4. 副标题 */
    .poem .subtitle {
      text-align: center;
      font-size: 14px;
      color: #9a8b73;
      letter-spacing: 2px;
      margin-bottom: 32px;
    }

    /* 5. 诗句正文：子代选择器，只管 .poem 直接下的 p */
    .poem > p {
      font-size: 20px;
      line-height: 2.2;
      letter-spacing: 3px;
      text-align: center;
    }

    /* 6. 作者署名：右对齐 + 斜体 */
    .poem .author {
      text-align: right;
      font-size: 15px;
      font-style: italic;
      color: #7d6b52;
      margin-top: 28px;
    }
  </style>
</head>
<body>

  <div class="poem">
    <div class="title">登高</div>
    <div class="subtitle">唐 · 杜甫</div>

    <p>风急天高猿啸哀，渚清沙白鸟飞回。</p>
    <p>无边落木萧萧下，不尽长江滚滚来。</p>
    <p>万里悲秋常作客，百年多病独登台。</p>
    <p>艰难苦恨繁霜鬓，潦倒新停浊酒杯。</p>

    <div class="author">—— 大历二年秋，夔州</div>
  </div>

</body>
</html>
```

这个案例值得记住的不是代码本身，而是**做事顺序**：

1. 先拆结构：标题 / 副标题 / 诗句 / 署名，各自是一个独立的块；
2. 再想定位：所有样式都挂在 `.poem` 下面，用后代、子代选择器锁住作用范围，绝不污染全局；
3. 最后才是调字号、行高、字间距这些「审美参数」。

> 一上来就调背景色和渐变，是新手最容易走偏的路。结构对了，样式才有地方落脚。

---

## 31 · 字体样式：color / family / size / style / weight

从这里开始进入「文字长什么样」的领域。CSS 里跟字体相关的属性，全都以 `font-` 开头（除了颜色 `color`）。

### ① color 文字颜色

```css
p {
  color: red;              /* 预定义颜色名：red / blue / tomato…… */
  color: #ff0000;          /* 十六进制，最常用，#f00 是简写 */
  color: rgb(255, 0, 0);   /* 三原色，0~255 */
  color: rgba(255, 0, 0, 0.5); /* 带透明度，0~1 */
}
```

实际项目里推荐用十六进制或 rgba：前者短，后者能控制透明度（做遮罩、hover 变淡特别好用）。

### ② font-family 字体族

```css
body {
  font-family: "Microsoft YaHei", "PingFang SC", "Helvetica Neue", Arial, sans-serif;
}
```

规则只有三条，但每条都会踩坑：

1. **从左往右找，找到哪个用哪个**——用户电脑上没有第一个字体，就退到第二个，这叫「字体回退」；
2. **字体名带空格必须加引号**：`"Microsoft YaHei"` 不加引号会被解析成三个单词，直接失效；
3. **最后一定要兜底一个通用族**：`serif`（衬线）、`sans-serif`（无衬线）、`monospace`（等宽）。

中英文混排的排版技巧是「英文在前、中文在后」：让英文用 Arial，中文自然回退到微软雅黑或苹方，两边各自好看。

### ③ font-size 字号

```css
.box { font-size: 16px; }    /* 固定像素，最常用也最直观 */
.box { font-size: 120%; }    /* 相对父元素的字号 */
.box { font-size: 1.2em; }   /* 相对当前元素（或父元素）的字号，会层层累乘 */
.box { font-size: 1rem; }    /* 相对根元素 html 的字号，不会累乘 */
```

- 浏览器默认 `16px`；
- 正文 `14~16px` 最舒服，标题 `20~32px`；
- `em` 会「套娃」（父元素 1.2em、子元素又 1.2em，实际是 1.44 倍），`rem` 不会，所以现在更推荐 `rem`。

### ④ font-style 倾斜

```css
.normal  { font-style: normal; }  /* 默认，不倾斜 */
.italic  { font-style: italic; }  /* 斜体，用字体自带的斜体字形 */
.oblique { font-style: oblique; } /* 强制把正体倾斜，没有斜体字形时才用 */
```

中文的 `italic` 效果很弱（很多中文字体没有真正的斜体字形，浏览器会「假斜」），所以中文一般不用这个属性做强调，用加粗或换颜色更好。

### ⑤ font-weight 字重

```css
.light { font-weight: 300; }
.normal { font-weight: 400; }  /* 等同 normal，默认值 */
.bold { font-weight: 700; }    /* 等同 bold */
.heavy { font-weight: 900; }
```

`100~900` 九档，但**字体本身得支持这么多字重才看得出来**。中文常用字体多数只有「常规 + 加粗」两档，写 300 和 500 视觉上没区别，写 900 也就跟 700 一样。

### ⑥ font 复合简写（重点）

`font` 是上面五个属性的「打包版」：

```css
/* 完整顺序：style variant weight size/line-height family */
font: italic bold 16px/1.8 "Microsoft YaHei", Arial, sans-serif;
```

三条必须记住的规则：

1. **`font-size` 和 `font-family` 是必填项**，少一个整条声明直接作废；
2. `line-height` 要写在 `font-size` 后面，用 `/` 连起来；
3. **简写会重置你没写的属性**——比如你只写了 `font: 16px Arial`，那 `font-weight`、`font-style` 全被重置成默认值，之前单独设的加粗会消失。

所以：**要么全写简写，要么老老实实分开写**，别混着来。

---

## 32 · 文本排版：对齐、缩进、装饰、行高、间距

字体属性管「字本身长什么样」，文本属性（`text-`）管「这些字怎么摆在容器里」。

### ① text-align 水平对齐

```css
h1 { text-align: center; }   /* 居中 */
p  { text-align: left; }     /* 左对齐，默认 */
.quote { text-align: right; }/* 右对齐 */
.article { text-align: justify; } /* 两端对齐，每行左右都顶格 */
```

一个关键认知：**`text-align` 让工具栏里的「行内内容」对齐，不会把块级盒子挪位置。**

比如给一个 `width: 300px` 的 `div` 设 `text-align: center`，居中的是里面的文字和 `inline` 元素，不是这个 `div` 自身。想让盒子居中，还得靠 `margin: 0 auto`（后面学布局时会细讲）。

### ② text-indent 首行缩进

```css
p {
  text-indent: 2em;   /* 缩进两个字符宽度，中文段落标配 */
}
```

- `2em` 跟着字号走：字号 16px 就缩 32px，字号 20px 就缩 40px，永远都是「两个字」；
- **只缩第一行**，第二行起顶格；
- 写负值可以做「悬挂缩进」，列表标记那种效果。

### ③ text-decoration 装饰线

```css
a { text-decoration: none; }             /* 去掉链接默认下划线，最常用 */
.price-old { text-decoration: line-through; } /* 删除线，原价 */
.mark { text-decoration: underline; }    /* 下划线 */
```

还能带样式和颜色：

```css
.emphasis {
  text-decoration: underline wavy red;  /* 波浪红下划线 */
  text-decoration-thickness: 2px;       /* 线粗细 */
  text-underline-offset: 4px;           /* 线和字的距离 */
}
```

### ④ line-height 行高

这是排版里最影响「舒服程度」的属性：

```css
p {
  line-height: 1.8;   /* 推荐：纯数字，等于字号的 1.8 倍，且会按子元素字号重新计算 */
  /* line-height: 32px; 固定像素，子元素字号变了容易错位 */
  /* line-height: 200%; 百分比，继承的是计算后的值，容易踩坑 */
}
```

关键公式（顺便回答一个老问题）：

> **行高 = 字号 + 上半行距 + 下半行距**
> **上半行距 = 下半行距 =（行高 − 字号）/ 2**

所以 `line-height: 1.5` 并不是「两行之间空出 1.5 倍字号」，两行之间真正的空白只有 **0.5 倍字号**——上下各 0.25 倍，两行拼起来正好 0.5 倍。这也是为什么设了 1.5 倍行高，看着还是「不怎么宽」。

**单行文字垂直居中**这个经典技巧就来自这个公式：把 `height` 和 `line-height` 设成同一个值，上下半行距自动均分，文字就正中间了。

```css
.btn {
  height: 44px;
  line-height: 44px;   /* 与 height 相等 → 垂直居中 */
  text-align: center;  /* 水平居中 */
}
```

### ⑤ letter-spacing 与 word-spacing

```css
.title { letter-spacing: 4px; }     /* 字间距，汉字和字母都生效 */
.en    { word-spacing: 8px; }       /* 词间距，只对单词之间的空格生效 */
```

中文排版想「拉开一点、高级感」，用 `letter-spacing`；英文想调整单词之间的空隙，才轮到 `word-spacing`。

---

## 33 · font 简写、继承性与综合案例

### ① 继承性：哪些属性能「传下去」

CSS 里的属性分两类：**可继承** 和 **不可继承**。

可继承的基本都是「和文字相关」的：

```css
/* 只要给 body 设了，全站文字默认都改了 */
body {
  font-family: "Microsoft YaHei", Arial, sans-serif;
  font-size: 16px;
  color: #333;
  line-height: 1.8;
  text-align: left;
}
```

- ✅ 可继承：`font-family` / `font-size` / `font-style` / `font-weight` / `color` / `line-height` / `text-align` / `text-indent` / `letter-spacing` 等；
- ❌ 不可继承：`width` / `height` / `margin` / `padding` / `border` / `background` / `display` / `position` / `float`。

理解这条规则，能省下大量重复代码：**统一字体、颜色、行高，写在 `body` 上一次就够**，不用每个选择器都抄一遍。

两个坑要记住：

1. **继承来的样式权重最低**，随时会被任何直接声明覆盖。所以别指望 `body { color }` 能压住某个元素自己写的 `color`；
2. **`<a>` 标签是继承的「破坏者」**：浏览器给 `a` 内置了 `color` 和 `text-decoration`，它的默认声明会盖掉从父元素继承的 `color`。想让链接跟随父级颜色，必须显式写：

```css
a {
  color: inherit;          /* 强制继承父级颜色 */
  text-decoration: none;   /* 顺手去下划线 */
}
```

### ② 综合案例：一张「文章卡片」

把今天所有知识点串一遍：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>文章卡片</title>
  <style>
    /* 全局：字体、颜色、行高一次搞定，靠继承传下去 */
    body {
      font-family: "Microsoft YaHei", Arial, sans-serif;
      font-size: 16px;
      color: #333;
      line-height: 1.75;
      background: #f7f8fa;
    }

    .card {
      width: 640px;
      margin: 60px auto;
      padding: 32px 36px;
      background: #fff;
      border-radius: 10px;
      box-shadow: 0 6px 24px rgba(0, 0, 0, .08);
    }

    /* 子代选择器：只管卡片自己里面的 h2 */
    .card > h2 {
      font: bold 24px/1.4 "Microsoft YaHei", sans-serif;
      color: #1f2d5a;
      text-align: center;
      letter-spacing: 2px;
      margin-bottom: 6px;
    }

    .card .meta {
      text-align: center;
      font-size: 13px;
      color: #9aa3b2;
      margin-bottom: 26px;
    }

    /* 后代选择器：卡片里所有段落统一缩进两字符 */
    .card p {
      text-indent: 2em;
      margin-bottom: 14px;
    }

    /* 相邻兄弟：h3 后面的第一段，当作导语特殊处理 */
    .card h3 + p {
      text-indent: 0;
      font-style: italic;
      color: #5a6b8c;
      border-left: 3px solid #2f3e7e;
      padding-left: 12px;
    }

    .card h3 {
      font-weight: 700;
      font-size: 18px;
      margin: 24px 0 10px;
    }

    .card .tag {
      display: inline-block;
      font-size: 12px;
      color: #2c6fb5;
      background: #eaf2fb;
      padding: 2px 10px;
      border-radius: 10px;
      letter-spacing: 1px;
    }
  </style>
</head>
<body>

  <div class="card">
    <h2>关系选择器到底解决了什么问题</h2>
    <div class="meta">2026-11-16 · 5 分钟阅读</div>

    <h3>一、从「点名」到「定位」</h3>
    <p>单个选择器只能点名某个元素，而真实页面是树状的。</p>
    <p>关系选择器让你按层级去定位，比如「.box 里面的 p」。</p>

    <h3>二、四个符号</h3>
    <p>空格代表后代，&gt; 代表子代，+ 代表相邻兄弟，~ 代表通用兄弟。</p>

    <span class="tag">CSS</span>
    <span class="tag">选择器</span>
  </div>

</body>
</html>
```

---

## 34 · 用 Xmind 梳理知识体系

这一节老师没讲新语法，而是带着用 Xmind 把这一阶段的知识画成一张图。

我今天整理出来的就是本文开头那张：

- **第一层**：关系选择器 / 综合案例 / 字体样式 / 文本排版 / 选择器补充 / 继承性 / 易错速查；
- **第二层**：每类下面的具体属性，比如字体样式展开成 color、font-family、font-size、font-style、font-weight、font 简写；
- **第三层**：每个属性的「取值 + 坑点 + 示例」。

梳理过程里最有价值的不是图本身，而是发现了两处知识缺口：

1. 我一直以为「层级选择器会让权重变高」，其实是各部分权重相加；
2. 我记不住 LVHA 的顺序，画到图上才意识到它是四个伪类的**书写顺序**，而不是四个独立知识点。

> 画思维导图不是为了好看，是为了逼自己回答「这一层的下一个节点是什么」。答不上来的地方，就是没学会的地方。

---

## 35 · 分组选择器与链接伪类选择器

### ① 分组选择器（并集选择器）

语法：多个选择器用 **逗号** 隔开，共享同一段样式。

```css
/* 三个标题共用一套样式 */
h1, h2, h3 {
  font-weight: bold;
  color: #1f2d5a;
}

/* 多个类也能一起写 */
.box, .card, .panel {
  background: #fff;
  border-radius: 8px;
}
```

两个必须注意的点：

1. **每一项都要写全**。`.box, card` 里的 `card`（少了点）会被当成标签名，不是类名，这是最常见的低级错误；
2. **一个选择器写错，整条规则可能整体失效**。因为选择器列表被视为一个整体，其中某项无效时，整条声明在多数情况下都不生效——所以写分组选择器时，改完一定要回头检查每一项。

另外，分组选择器里每个选择器的**权重是各自独立计算**的，不会互相影响：

```css
/* .title 是 0,0,1,0；h2 是 0,0,0,1，各算各的 */
.title, h2 { color: red; }
```

### ② 链接伪类选择器与 LVHA 顺序

链接（`<a>`）有四种状态，对应四个伪类：

| 伪类 | 状态 | 说明 |
| --- | --- | --- |
| `a:link` | 未访问过 | 默认状态 |
| `a:visited` | 已访问过 | 浏览器记录过就算 |
| `a:hover` | 鼠标悬停 | 指针停在上面 |
| `a:active` | 按下未松开 | 鼠标左键按住的那一瞬 |

写链接样式，**顺序必须按 L → V → H → A**（业界口诀叫 **LoVe HAte**，爱恨顺序）：

```css
/* 顺序写对，四种状态才都能生效 */
a:link    { color: #333; text-decoration: none; }
a:visited { color: #8a8a8a; }
a:hover   { color: #e3544b; text-decoration: underline; }
a:active  { color: #b03027; }
```

为什么顺序不能乱？因为这四个伪类**权重完全相同**（都是 0,0,1,1）。权重相同时，**后写的覆盖先写的**。如果你把 `:hover` 写在 `:visited` 前面，已访问的链接悬停时，颜色就会被后面的 `:visited` 盖掉——hover 效果「消失」了。

还有一个冷知识：`:visited` 能改的属性被浏览器严格限制（基本只能改颜色），这是为了防止网站通过「链接变没变灰」来偷窥你浏览过什么。

### ③ 用户行为伪类：`:hover` 与 `:focus`

这两个管的是「用户对这个元素做了什么」。

```css
/* 鼠标悬停：卡片浮起来 */
.card {
  transition: transform .25s, box-shadow .25s;
}
.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 10px 30px rgba(0, 0, 0, .12);
}

/* 获得焦点：输入框高亮 */
input {
  border: 1px solid #d5dae5;
  outline: none;
}
input:focus {
  border-color: #2c6fb5;
  box-shadow: 0 0 0 3px rgba(44, 111, 181, .15);
}
```

三个细节：

1. **冒号不能漏**：`hover` 是伪类，必须写 `:hover`；写成 `hover` 会被当成一个叫 hover 的标签名；
2. **不要加括号**：`:hover` 不是函数，写成 `:hover()` 属于非法选择器；
3. **`:focus` 是键盘可达性的关键**。用 Tab 键在页面间跳转时，`:focus` 提示用户「现在在哪个控件上」，所以做设计时不要无脑 `outline: none`，最好换成更明显的边框或阴影。

小提醒：`hover` 在触屏设备上会有「粘滞」现象——点一下之后状态卡住不消失。正式项目里通常配合媒体查询，只在有鼠标的设备上启用 hover 效果：

```css
@media (hover: hover) {
  .card:hover { transform: translateY(-4px); }
}
```

---

## 36 · 今日完整示例：一份可运行的练习页

把 28~36 节所有东西塞进一个文件，方便随时打开复习：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>CSS 28~36 节综合练习</title>
  <style>
    /* ---------- 全局 ---------- */
    * { margin: 0; padding: 0; box-sizing: border-box; }

    body {
      font-family: "Microsoft YaHei", Arial, sans-serif;
      font-size: 16px;
      line-height: 1.75;
      color: #333;
      background: #f6f7fb;
      padding: 40px 16px;
    }

    .wrap { width: 720px; margin: 0 auto; }

    /* ---------- 卡片 1：关系选择器 ---------- */
    .demo {
      background: #fff;
      border-radius: 10px;
      padding: 26px 30px;
      margin-bottom: 26px;
      box-shadow: 0 4px 18px rgba(0, 0, 0, .06);
    }

    /* 子代：只改直接子元素标题 */
    .demo > h2 {
      font: bold 20px/1.5 "Microsoft YaHei", sans-serif;
      color: #1f2d5a;
      letter-spacing: 1px;
      margin-bottom: 12px;
    }

    /* 后代：demo 里所有段落 */
    .demo p { text-indent: 2em; color: #555; }

    /* 相邻兄弟：h3 后面紧挨着的 p 做导语 */
    .demo h3 + p {
      text-indent: 0;
      border-left: 3px solid #e3544b;
      padding-left: 10px;
      color: #8a4a45;
    }

    /* 通用兄弟：列表之间加分割线，第一项除外 */
    .list { list-style: none; margin-top: 12px; }
    .list li { padding: 10px 0; }
    .list li + li { border-top: 1px dashed #e6e9f2; }

    /* ---------- 卡片 2：字体与文本 ---------- */
    .type p:first-child { font-weight: 700; }
    .type .big   { font-size: 22px; }
    .type .ital  { font-style: italic; }
    .type .wide  { letter-spacing: 6px; }
    .type .strike{ text-decoration: line-through; color: #aaa; }

    /* ---------- 卡片 3：链接与伪类 ---------- */
    .links a:link    { color: #2c6fb5; text-decoration: none; }
    .links a:visited { color: #8a8a8a; }
    .links a:hover   { color: #e3544b; text-decoration: underline; }
    .links a:active  { color: #b03027; }

    .btn {
      display: inline-block;
      height: 42px;
      line-height: 42px;      /* 与 height 相等 → 垂直居中 */
      padding: 0 24px;
      background: #2f3e7e;
      color: #fff;
      border: none;
      border-radius: 21px;
      font-size: 15px;
      letter-spacing: 2px;
      cursor: pointer;
      transition: background .25s;
    }
    .btn:hover { background: #4257ab; }
    .btn:focus { outline: 3px solid rgba(47, 62, 126, .3); outline-offset: 3px; }

    input {
      width: 240px;
      padding: 10px 12px;
      border: 1px solid #d5dae5;
      border-radius: 8px;
      font-size: 15px;
      outline: none;
    }
    input:focus {
      border-color: #2c6fb5;
      box-shadow: 0 0 0 3px rgba(44, 111, 181, .15);
    }
  </style>
</head>
<body>

<div class="wrap">

  <!-- 卡片 1 -->
  <div class="demo">
    <h2>一、关系选择器练习</h2>
    <p>这一段被 .demo p 后代选择器统一加了 2em 首行缩进。</p>

    <h3>导语段落</h3>
    <p>我是 h3 后面紧挨着的 p，被 h3 + p 单独处理，不再缩进。</p>
    <p>我是第二个 p，符合 .demo p，但不符合 h3 + p。</p>

    <ul class="list">
      <li>列表第一项：上面没有分割线</li>
      <li>列表第二项：上面有虚线，来自 li + li</li>
      <li>列表第三项：同样有虚线</li>
    </ul>
  </div>

  <!-- 卡片 2 -->
  <div class="demo type">
    <h2>二、字体与文本练习</h2>
    <p>普通段落，默认 16px，行高 1.75。</p>
    <p class="big">这一段 22px，明显更大。</p>
    <p class="ital">这一段是斜体（italic）。</p>
    <p class="wide">字间距拉开 6px，标题感更强。</p>
    <p class="strike">这一段加了删除线，像划掉的原价。</p>
  </div>

  <!-- 卡片 3 -->
  <div class="demo">
    <h2>三、伪类与交互练习</h2>
    <p class="links">
      <a href="#">把鼠标移上来，我会变红并出现下划线</a>
    </p>
    <p style="text-indent:0; margin-top:16px;">
      <button class="btn">悬停试试按钮</button>
    </p>
    <p style="text-indent:0; margin-top:16px;">
      <input type="text" placeholder="点我试试 :focus 高亮">
    </p>
  </div>

</div>

</body>
</html>
```

---

## 今日踩坑小结

1. **空格 = 后代，`>` = 子代，`+` = 相邻兄弟，`~` = 通用兄弟**——四个符号四个意思，别混；
2. **兄弟选择器只能往后选**，永远选不到前面的兄弟，这是 CSS 的单向性；
3. **关系选择器不会额外加权重**，`div p` 是 0,0,0,2 是因为它有两个标签；
4. **`font-family` 里带空格的字体名必须加引号**，最后记得兜底 `serif` / `sans-serif`；
5. **`font` 简写必须带 `font-size` 和 `font-family`**，少一个整条失效；而且它会重置未写的属性；
6. **`line-height: 1.5` 不是「行间空 1.5 倍字」**，真正的行间空白只有 0.5 倍字号（上下各 0.25 倍）；
7. **单行垂直居中 = `height` 与 `line-height` 相等**，这是行高公式的自然结果；
8. **`text-align` 只让行内内容对齐**，不会把块级盒子本身挪动；
9. **继承的是文字类属性，盒模型和布局属性不继承**；`<a>` 的默认 color 会破坏继承，需要 `color: inherit`；
10. **链接伪类顺序 LoVe HAte（L→V→H→A）**，四个伪类权重相同，写反就被后写的盖掉；
11. **`:hover` 冒号不能漏、括号不能加**，触屏设备上配 `@media (hover: hover)` 更稳；
12. **分组选择器每一项都要写全**，`.box, card` 这种少个点的写法会出大问题。

明天该往「CSS 三大特性 + 盒子模型」走了——层叠、继承今天已经开了个头，盒子模型才是真正决定页面长什么样的地基 💪

---

## 附：今天的思维导图

- 在线预览：本页顶部图片
- Xmind 源文件：[下载 css-relations-font-text.xmind](/files/2026-11-16-css-relations-font-text.xmind)

导入方式：Xmind 打开 → 文件 → 导入 → 选择该 `.xmind` 文件，即可在「关系选择器」根节点下继续补充。

## 自测清单

- [ ] 不看笔记，说出 `.box p`、`.box > p`、`.box + p`、`.box ~ p` 四者的区别
- [ ] 写出「相邻列表项之间加分割线，第一项不加」的一行 CSS
- [ ] 解释 `line-height: 1.5` 时两行文字之间的空白是多少倍字号
- [ ] 说出 `font` 简写里哪两个值不能省略，省略了会怎样
- [ ] 说出可继承属性和不可继承属性各三类
- [ ] 默写四种链接伪类的正确顺序，并解释为什么不能乱
- [ ] 解释为什么 `<a>` 标签不继承父级的 `color`
