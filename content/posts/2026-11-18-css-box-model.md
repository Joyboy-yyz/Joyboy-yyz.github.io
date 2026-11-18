---
title: "2026-11-18 学习日志：CSS 盒子模型 —— 边框、圆角、内边距、外边距与居中"
date: 2026-11-18T12:00:00+08:00
draft: false
tags: ["CSS", "前端", "盒子模型", "border", "border-radius", "padding", "margin", "box-sizing", "居中", "margin塌陷", "学习日志"]
categories: ["学习笔记"]
summary: "Pink 老师前端课第 41~44 节：先把「页面里每个元素都是一个盒子」这件事说透——content / padding / border / margin 四层结构、真实尺寸怎么算；再拆 border 的三要素与分方向写法、border-radius 用圆切角的几何原理与 50% 为什么是圆；然后是 padding 的四值简写与「撑大盒子」的连锁反应，用 box-sizing: border-box 一次性解决；最后是 margin 的四值、负值、水平居中 margin: 0 auto 的成立条件，以及兄弟合并、父子塌陷这两个最阴的坑。"
---

> 今天的知识地图已经更新到 Xmind（`2026-11-18-css-box-model.xmind`），文末可直接下载源文件导入 Xmind 继续编辑。

![CSS 盒子模型四层结构与尺寸公式](/images/css-box-model.png)

---

## 41 · 盒子模型：页面里每个元素都是一个「盒子」

### 一句话理解

> **你写的每一个 div、每一个 p、每一张图片，浏览器都把它当成一个矩形盒子来摆放。盒子模型就是告诉你：这个矩形由哪几层拼起来，每一层改哪个属性、会不会改变尺寸。**

从内到外一共四层：

| 层级 | 属性 | 是什么 | 大白话 | 会不会撑大盒子 |
|---|---|---|---|---|
| ① 内容区 | `width` / `height` | 文字、图片真正占的区域 | 「盒子里装的东西」 | 基准，其他都围着它长 |
| ② 内边距 | `padding` | 内容到边框之间的**填充** | 「东西和盒壁之间的泡沫」 | **会**（除非 border-box） |
| ③ 边框 | `border` | 盒子自己的**外壁** | 「盒子的壳，有厚度」 | **会** |
| ④ 外边距 | `margin` | 盒子与**其他盒子**之间的空隙 | 「盒子之间的空气，透明的」 | **不会**，只把别人推远 |

⚠️ 最容易混的一组：**padding 是「盒子的内壁到内容」的距离，margin 是「盒子和邻居」的距离**。padding 长在盒子里、背景能盖到；margin 长在盒子外、完全透明。

### 尺寸公式（先背下来，后面全靠它）

```
内容宽度   = width
盒子实际宽度 = width + padding-left + padding-right + border-left + border-right
盒子占位宽度 = 盒子实际宽度 + margin-left + margin-right
实际高度同理
```

举个具体数：`width: 300px; padding: 10px; border: 2px solid;` 的盒子

- 内容宽 300
- 实际宽 300 + 10×2 + 2×2 = **324px**
- 如果再有 `margin: 0 20px`，它在页面上横着吃掉 324 + 40 = **364px**

很多「两个 50% 宽的盒子放一行结果换行了」「设了 100px 结果量出来 120px」的诡异现象，全是这条公式没算清楚。

### 三种盒子的脾气不一样

| 盒子类型 | display | 宽度 | 垂直 margin | 能不能并排 |
|---|---|---|---|---|
| 块级 | `block`（div/p/h/li） | 默认占满父级一行 | **会合并**（见 44 节） | 不能，独占一行 |
| 行内 | `inline`（span/a/strong） | 由内容撑开，设 width 无效 | 上下 margin **不占位** | 能并排 |
| 行内块 | `inline-block`（img/input） | 可设宽高 | 正常生效 | 能并排 |

> 大白话：块级是「霸道总裁，一行只站我一个」；行内是「老实人，挤一挤也能过」；行内块是「有个子的老实人，能设宽高但还能并排」。

---

## 41（续）· 边框 border：三要素缺一不可

### 三个子属性

```css
border-width: 2px;        /* 粗细：px / thin / medium / thick */
border-style: solid;      /* 线型：solid / dashed / dotted / double / none */
border-color: #333;       /* 颜色：不写默认取 color 的值（currentColor） */
```

⚠️ **最常见的「边框不出来」**：只写了 `border-width: 2px; border-color: red;`，忘了 `border-style`。默认值是 `none`，**线型为 none 时整条边框直接不渲染**，宽高都白写。

### 复合写法（工作里 99% 用这个）

```css
border: 2px solid #333;        /* 顺序随便，但习惯 width style color */
```

### 分方向 / 分属性

```css
/* 只要一条边 */
border-top: 3px dashed #e74c3c;
border-bottom: 1px solid #ddd;

/* 四条边不同：四个方向各写一条，别指望一个 border 搞定 */
```

可拆的粒度一共 12 个：`border-top-width / border-top-style / border-top-color` × 4 个方向，一般只在做「只有上边框变色」这种细节时才拆。

### 边框算进尺寸（重点）

border 属于盒子内部，**会把盒子撑大**：`width: 100px; border: 5px solid` → 实际 110px。所以：

- 想「总宽固定 200px 且带边框」→ 要么 `width: 190px`，要么直接 `box-sizing: border-box`（43 节讲）；
- 相邻两个带边框的盒子贴在一起，中间会出现 **2px 双线**。

### 表格细线：`border-collapse`

```css
table { border-collapse: collapse; }   /* 相邻边框合并成一条 */
td, th { border: 1px solid #ccc; }
```

- 默认 `border-collapse: separate`：每个单元格各画一圈，相邻处 **2px**；
- `collapse` 后合并成 1px，这才是「细线表格」。

### 彩蛋：用 border 画三角形（理解边框是「梯形」的）

```css
.triangle {
  width: 0; height: 0;
  border: 20px solid transparent;
  border-top-color: #f39c12;   /* 只有上边框有颜色 → 一个尖朝下的三角 */
}
```

为什么能画出来？因为**边框在四个角是 45° 斜接的**——把盒子的内容区压成 0，四条边框就变成四个三角形拼在一起。理解了这一点，才算真懂 border 的几何模型。

---

## 42 · 圆角边框 border-radius 与它的原理

### 基础用法

```css
.box { border-radius: 10px; }       /* 四个角一起圆 */
.btn { border-radius: 8px 8px 0 0; }/* 左上 右上 右下 左下（顺时针） */
```

四值顺序和 padding / margin 不一样：**圆角是「顺时针」从左上角开始**，记成「左上 → 右上 → 右下 → 左下」。

单角专用属性名：

```css
border-top-left-radius: 12px;
border-top-right-radius: 12px;
```

### 原理：圆角是「拿圆去切掉直角」

把它想成三步：

1. 在盒子的**每个角**，画一个半径 r 的圆，圆心放在「距两条边都是 r」的位置；
2. 用这段**四分之一圆弧**替换原来的直角；
3. r 越大，切掉的肉越多，角就越圆。

```
r 小：  ┐         r = 边长一半：成为圆       r > 边长一半：被规则强制缩小
       ╯
```

所以：

- **正方形 + `border-radius: 50%` = 正圆**。因为 50% 半径正好等于边长一半，四个角在中点相切，直角全被切光；
- **`border-radius: 999px` 和 `100px` 在 200×200 的盒子上长得一样**。规范规定：一条边上两个圆角半径之和超过边长时，所有半径**按比例缩小**，避免圆弧互相重叠。所以「写个超大值保险」是可行的，浏览器会帮你兜底；
- **长方形 + `50%` ≠ 胶囊按钮**，而是**四角椭圆**。因为百分比圆角：水平半径按**宽度**算，垂直半径按**高度**算。200×48 的按钮，`50%` = 水平 100px、垂直 24px → 两端是半椭圆。

> 想要「两端半圆的胶囊按钮」：`border-radius: 24px`（= 高度的一半），不是 50%。

### 进阶：斜杠写法（椭圆角）

```css
/* 水平半径 / 垂直半径 */
border-radius: 50px / 20px;

/* 完整 8 值：四个角的水平半径 / 四个角的垂直半径 */
border-radius: 10px 20px 30px 40px / 5px 10px 15px 20px;
```

斜杠前是「横着多圆」，斜杠后是「竖着多圆」，靠它才能做出「叶子形」「水滴形」这类造型。

### 圆角会不会裁掉子元素？

**不会自动裁。** 背景色会跟着圆角走（`background-clip` 默认 `border-box`，但你看到的是被圆角轮廓裁过的视觉），但**子元素超出圆角的部分仍然方方正正露在外面**。

```css
.card {
  border-radius: 16px;
  overflow: hidden;   /* 让子元素（图片、色块）也被圆角裁掉 */
}
```

⚠️ 这是做「圆角卡片里放通栏图片」时最经典的一坑：忘了 `overflow: hidden`，图片的直角会顶破圆角。

### 尺寸不会变

`border-radius` 只改**轮廓的视觉形状**，不参与盒子的宽高计算，也不会减少占位。100×100 加了 `border-radius: 50%` 之后，仍是占 100×100 的位置，只是四个角「看起来没了」。

---

## 43 · 内边距 padding：内容与盒壁之间的呼吸

### 作用

给内容和边框之间留出空隙，让文字不贴着边框。**背景色 / 背景图会铺满 padding 区域**——这也是 padding 和 margin 最直观的区别：padding 是「盒子内部的空间」，看得见背景；margin 是「盒子外面的空气」，什么都没有。

### 四值简写（必背）

```css
padding: 20px;                 /* 上下左右全 20 */
padding: 10px 20px;            /* 上下 10，左右 20 */
padding: 10px 20px 30px;       /* 上 10，左右 20，下 30 */
padding: 10px 20px 30px 40px;  /* 上 右下 左（顺时针：上→右→下→左） */
```

记忆口诀：**「缺省就对称」**——少一个值就照对面的值抄。

### 分方向

```css
padding-top: 12px;
padding-right: 20px;
padding-bottom: 12px;
padding-left: 20px;
```

### 核心连锁反应：padding 会撑大盒子

```css
.box {
  width: 300px;
  padding: 0 20px;   /* 左右各 20 */
  border: 1px solid #ccc;
}
/* 实际宽度 = 300 + 20×2 + 1×2 = 342px */
```

三种解法：

1. **手动减**：`width: 258px` —— 改一次 padding 就得重算，维护地狱；
2. **不设 width，靠 padding 撑**：适合「自适应内容宽度」的按钮、标签；
3. **`box-sizing: border-box`** ← 现代项目标准答案。

### `box-sizing`：让 width 变成「总宽度」

```css
*, *::before, *::after { box-sizing: border-box; }
```

| 值 | width 指的是 | 计算公式 |
|---|---|---|
| `content-box`（默认） | 只有内容区 | 实际 = width + padding + border |
| `border-box` | **内容 + padding + border** | 实际 = width；内容区自动被挤压 |

切换之后同一个 `width: 300px; padding: 0 20px; border: 1px`：

- content-box：实际 **342px**
- border-box：实际 **300px**，内容区只剩 258px

> 大白话：content-box 是「我 300 斤，加上衣服和背包一共 342 斤」；border-box 是「连人带衣服带包，一共 300 斤」。做布局当然希望「总共多宽就写多宽」，所以几乎所有 reset / 框架第一行都是 `box-sizing: border-box`。

### padding 的硬规则与冷知识

- **不能为负**，写了也无效（负值只有 margin 和定位能玩）；
- **百分比 padding / margin 全都按「包含块的宽度」算**，包括 `padding-top`。父级宽 500，`padding-top: 10%` 就是 50px，不是高度的 10%——这是老规范留下的坑，做「等比占位 / 正方形」时常用它；
- **行内元素的上下 padding 不撑开行高**：`span { padding: 20px 0 }` 背景色会画出去，但不会把上下行推开 → 视觉上「压在别的文字上」。行内元素要留上下空间，改成 `inline-block` 或加行高；
- padding 写在简写里最容易误伤：`padding: 10px 0` 会**把左右 padding 清零**，原来设的左右值就没了。

### 什么时候该用 padding，什么时候该用 margin？

| 场景 | 用哪个 | 原因 |
|---|---|---|
| 文字到边框的距离 | padding | 属于盒子内部 |
| 按钮点击区域变大 | padding | 扩大的是盒子自身 |
| 两个卡片之间的距离 | margin | 是盒子与盒子的关系 |
| 卡片里图片和标题的间距 | 两者皆可，看语义 | 同一盒子内部 → padding 更合理 |

---

## 44 · 外边距 margin：把盒子推开，以及「居中」这件事

### 作用与写法

margin 是盒子**外面**的透明空间，用来推远邻居。四值简写规则和 padding 完全一致：

```css
margin: 20px;                  /* 四周 20 */
margin: 0 auto;                /* 上下 0，左右自动（水平居中） */
margin: 10px 20px;             /* 上下 10，左右 20 */
margin: -10px 0 0 0;           /* 可以负值 */
```

**margin 不撑大盒子**，但会占用页面空间：量一个盒子在页面上「吃掉多少」，必须把 margin 也算进去。

### 负 margin 能干什么

- `margin-top: -20px`：整体上移，压住上面的元素（做「头像叠在封面上」常用）；
- `margin-left: -1px`：让两个带边框的盒子边框重叠，消掉那条双线；
- `margin-right: -20px`：让内容突破父级 padding 的边界（做「贴边横滑」常用）。

⚠️ 负 margin 是「用位置换遮挡」的暴力手段，能用 flex/grid 的 `gap` 解决就优先用 gap。

### 水平居中：`margin: 0 auto` 的成立条件

```css
.box {
  width: 600px;      /* ① 必须有明确宽度 */
  margin: 0 auto;    /* ② 左右设为 auto */
}
```

**三个条件缺一不可**：

1. 元素是**块级**（`display: block/table` 等），行内元素 `margin: 0 auto` 无效；
2. 有**确定的宽度**（写了 width，或者有 max-width），否则块级元素默认占满一行，没有剩余空间可分配；
3. 处于**普通文档流**中（浮动、绝对定位元素不适用这套规则）。

原理：浏览器把「父级剩余宽度」平均分给左右两个 `auto`，所以左右相等 → 居中。

```
[← auto →][   600px   ][← auto →]
```

**垂直方向 `margin: auto` 不居中**：块级元素的上下 margin 设 auto 会算成 0（普通流里高度由内容决定，没有「剩余高度」可分）。垂直居中交给 `flex` 的 `align-items: center`，或者绝对定位 + `transform`。

### 其他居中姿势（按场景选）

```css
/* ① 块级盒子水平居中 */
.box { width: 600px; margin: 0 auto; }

/* ② 行内元素 / 文字 / 行内块（img、span）水平居中：给父级 */
.parent { text-align: center; }

/* ③ 绝对定位居中（不要求有宽度） */
.center {
  position: absolute;
  left: 50%; top: 50%;
  transform: translate(-50%, -50%);
}

/* ④ 弹性布局：最通用，横竖都能中 */
.parent { display: flex; justify-content: center; align-items: center; }
```

⚠️ `text-align: center` 只影响**行内内容**，对一个块级子 div 无效；想让块级子元素居中，还是 `margin: 0 auto`。

### 大坑一：兄弟元素 margin 合并（取最大值，不是相加）

```css
.a { margin-bottom: 30px; }
.b { margin-top: 20px; }
/* 两者之间实际是 30px，不是 50px */
```

垂直方向相邻的两个块级盒子，margin 会**合并成一个**，值取两者最大值（有负值时按「正负相加、取绝对值最大」的规则计算）。这叫 **margin collapsing（外边距合并）**。

- 只发生在**垂直**方向，左右 margin 从不合并；
- 只发生在**普通流块级**元素之间；浮动、绝对定位、行内块、flex/grid 子项**都不合并**（这也是为什么 flex 布局里 gap 用起来那么省心）。

### 大坑二：父子 margin 塌陷（子的 margin 跑到了父身上）

```html
<div class="father">
  <div class="son"></div>
</div>
```
```css
.father { background: #eee; }
.son { margin-top: 50px; background: #3498db; }
```

结果：**父盒子被一起往下推了 50px，父盒子顶部出现一条 50px 的空白**，而子盒子紧贴父盒子顶部。

原因：父元素没有 border-top / padding-top，子元素的 margin-top 会和父元素的 margin-top **合并**，于是「子的外边距」变成了「父的外边距」。

四种解法，任选其一即可：

```css
.father { overflow: hidden; }       /* ① 触发 BFC，最常用 */
.father { display: flow-root; }     /* ② 现代写法，专为这个场景设计，无副作用 */
.father { padding-top: 1px; }       /* ③ 加内边距（会改变视觉，慎用） */
.father { border-top: 1px solid transparent; }  /* ④ 加透明边框 */
```

> 顺带一个判断技巧：**「父盒子莫名多了个上边距」「高度少了 50px」→ 九成是父子塌陷**；打开 DevTools 看父盒子的 margin，如果凭空多出来值，就是它。

---

## 综合 Demo：一张「卡片 + 居中列表」，四件套全上

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>盒子模型综合练习</title>
<style>
  /* 0. 全局：把 width 变成"总宽度"，从此不再算 padding 的账 */
  *, *::before, *::after { box-sizing: border-box; }

  body {
    margin: 0;                      /* 清掉浏览器默认 8px */
    padding: 40px 20px;
    background: #f5f7fa;
    font: 15px/1.7 "Microsoft YaHei", sans-serif;
    color: #333;
  }

  /* 1. 容器：定宽 + 水平居中（margin: 0 auto） */
  .wrap {
    width: 760px;
    margin: 0 auto;
  }

  /* 2. 卡片：圆角 + 边框 + 内边距 + 裁掉子元素溢出 */
  .card {
    background: #fff;
    border: 1px solid #e3e6eb;
    border-radius: 14px;
    padding: 22px 26px;
    margin-bottom: 24px;            /* 卡片之间的距离用 margin */
    overflow: hidden;               /* 关键：让圆角真正裁到内部 */
    transition: box-shadow .25s, transform .25s;
  }
  .card:hover {
    box-shadow: 0 8px 24px rgba(0, 0, 0, .1);
    transform: translateY(-2px);
  }

  /* 3. 卡片内的图片：通栏，左右用负 margin 顶到卡片边缘 */
  .card img {
    display: block;
    width: calc(100% + 52px);       /* 抵消左右 padding 26×2 */
    margin: -22px -26px 18px;       /* 上 -22 抵消 padding-top */
    border-radius: 0;
  }

  /* 4. 标题：只保留下边框做分隔 */
  .card h2 {
    margin: 0 0 10px;
    font-size: 18px;
    color: #1f2d3d;
    padding-bottom: 8px;
    border-bottom: 1px dashed #e3e6eb;
  }

  /* 5. 标签：行内块 + 胶囊圆角（半径 = 高度一半） */
  .tag {
    display: inline-block;
    height: 24px;
    line-height: 24px;
    padding: 0 12px;
    margin-right: 8px;
    font-size: 12px;
    color: #fff;
    background: #4c8bf5;
    border-radius: 12px;            /* 不是 50%，因为它是长方形 */
  }

  /* 6. 头像：正方形 + 50% 得到正圆，负 margin 上移压边 */
  .avatar {
    width: 64px; height: 64px;
    border-radius: 50%;
    border: 3px solid #fff;
    box-shadow: 0 2px 8px rgba(0,0,0,.15);
    margin: -46px 0 0 0;            /* 负 margin：往上顶，压在图片上 */
    position: relative;
  }

  /* 7. 一行三个等宽卡片：用 flex + gap，避开 margin 合并 */
  .row {
    display: flex;
    gap: 16px;
    margin-top: 24px;
  }
  .row .mini {
    flex: 1;                        /* 三等分，不用自己算宽度 */
    background: #fff;
    border: 1px solid #e3e6eb;
    border-radius: 12px;
    padding: 18px;
  }
</style>
</head>
<body>

<div class="wrap">
  <div class="card">
    <img src="https://picsum.photos/760/240" alt="封面">
    <img class="avatar" src="https://picsum.photos/64" alt="头像">
    <h2>盒子模型四件套</h2>
    <p>width 是内容区，padding 是内壁泡沫，border 是壳，margin 是盒子外面的空气。</p>
    <span class="tag">border</span>
    <span class="tag">padding</span>
    <span class="tag">margin</span>
    <span class="tag">border-radius</span>
  </div>

  <div class="row">
    <div class="mini"><h3>内容</h3><p>真正放文字的地方</p></div>
    <div class="mini"><h3>内边距</h3><p>背景会铺到这里</p></div>
    <div class="mini"><h3>外边距</h3><p>完全透明不占背景</p></div>
  </div>
</div>

</body>
</html>
```

这个 demo 里用到了：`box-sizing` 重置、`margin: 0 auto` 居中、卡片圆角 + `overflow: hidden`、负 margin 让图片顶边、胶囊按钮（半径 = 高度一半）、正方形 50% 变圆、`flex + gap` 避开 margin 合并。**这六个点是今天全部考点。**

---

## 今日踩坑小结（12 条，都是真会踩的）

1. **写了 border 宽度和颜色，边框却不出现** → 漏了 `border-style`，默认 `none` 不渲染。
2. **设了 `width: 300px`，量出来 342px** → padding 和 border 撑大了盒子，用 `box-sizing: border-box`。
3. **两个 50% 的盒子放一行换行了** → 各加了 padding / border，实际超过 50%；用 border-box 或 `flex: 1`。
4. **表格边框是双线** → 忘了 `border-collapse: collapse`。
5. **圆角卡片里的图片顶破了角** → 忘写 `overflow: hidden`。
6. **长方形的按钮写 `border-radius: 50%` 变成椭圆角** → 胶囊按钮应该写「高度的一半」，如 `24px`。
7. **`border-radius: 999px` 和 `100px` 效果一样** → 半径和超过边长会被按比例压缩，不是 bug。
8. **`padding: 10px 0` 把左右内边距清零了** → 简写会覆盖全部四个方向。
9. **`margin: 0 auto` 不居中** → 元素是行内（没写 display）、或者没写宽度、或者被浮动了。
10. **两个盒子间距本该 50px，实际只有 30px** → 垂直 margin 合并取最大值；用 flex + gap 最省事。
11. **父盒子莫名被推下去一块** → 父子 margin 塌陷，给父级 `overflow: hidden` 或 `display: flow-root`。
12. **`span` 加了上下 padding，结果压住了别的文字** → 行内元素上下 padding 不撑开行高，改成 `inline-block`。

> 排查顺序建议：先 `outline: 1px solid red` 看盒子真实边界（outline 不占空间、不改尺寸，比 border 更适合调试），再在 DevTools 的 Computed 面板里看那个四层彩色方块图——它写的就是盒子模型本身。

---

## 实战练习（由易到难，30 分钟内能做完）

**① 新闻列表卡片（10 分钟）**
做一张 600px 宽、水平居中的卡片：白底、1px 浅灰边框、12px 圆角、内边距 20px 30px，标题下方一条虚线下边框，正文 3 行。要求「卡片总宽正好 600px」，验证方式：DevTools 量盒子实际宽度。

**② 三个等宽商品卡（10 分钟）**
一行 3 个卡片，间距 16px，容器 960px 居中。两种写法都试一遍：
- 用 `width: 33.33%` + `margin-right: 16px`（会溢出，体会一下为什么）；
- 用 `display: flex; gap: 16px;` + `flex: 1`（观察差异）。

**③ 个人主页头部（10 分钟）**
顶部 240px 高的通栏封面图，头像 80×80 圆形、白色 4px 边框、压在封面下边缘（用负 margin 实现），下面跟昵称、简介。做完再故意把父级 `overflow: hidden` 删掉，看圆角处发生了什么。

---

## 自测清单

- [ ] 默画盒子模型四层，并说出每层对应的 CSS 属性
- [ ] 一个 `width: 200px; padding: 10px; border: 2px solid` 的盒子，实际宽多少？占位宽多少（再加 `margin: 0 5px`）？
- [ ] `border-style` 漏写会发生什么？为什么
- [ ] `border-collapse: collapse` 解决的是什么问题
- [ ] 说清 border 为什么能画出三角形（45° 斜接）
- [ ] `border-radius` 四值的顺序，和 `padding` 四值顺序有什么不同
- [ ] 为什么正方形 `50%` 是圆，而长方形 `50%` 是椭圆角
- [ ] 写出「两端半圆胶囊按钮」的两种写法（固定高度 / 百分比）
- [ ] `border-radius` 会不会改变盒子占位尺寸
- [ ] `content-box` 和 `border-box` 下 `width` 分别指什么
- [ ] 为什么 `padding-top: 10%` 是按**宽度**算的
- [ ] 行内元素设上下 padding 会撑开行高吗
- [ ] `margin: 0 auto` 成立的三个条件
- [ ] 为什么上下 `margin: auto` 不能垂直居中
- [ ] 兄弟 margin 合并取最大值还是相加？哪些情况不合并
- [ ] 父子 margin 塌陷的现象、原因、四种解法
- [ ] 为什么调试盒子边界推荐 `outline` 而不是 `border`

---

## 附：今天的思维导图

- Xmind 源文件：[下载 2026-11-18-css-box-model.xmind](/files/2026-11-18-css-box-model.xmind)
- 结构图：[盒子模型四层与尺寸公式](/images/css-box-model.png)

导入方式：Xmind 打开 → 文件 → 导入 → 选择 `.xmind` 文件。这份图谱把 41~44 节拆成「四层组成 / 边框 / 圆角 / 内边距 / 外边距 / 尺寸计算 / 坑位清单」七个分支，复习时按分支展开即可。

> 盒子模型是 CSS 布局的地基：**以后所有「为什么这里多了 8px」「为什么这个盒子对不齐」的问题，答案几乎都在这四层里**。下一篇进浮动与定位，盒子就该开始「脱离文档流」了 💪
