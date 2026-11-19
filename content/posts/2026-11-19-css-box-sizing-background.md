---
title: "2026-11-19 学习日志：CSS 盒子模型（二）—— box-sizing 尺寸计算、background 背景全解与渐变"
date: 2026-11-19T12:00:00+08:00
draft: false
tags: ["CSS", "前端", "盒子模型", "box-sizing", "background", "背景", "background-size", "background-position", "background-attachment", "渐变", "linear-gradient", "文字渐变", "学习日志"]
categories: ["学习笔记"]
summary: "Pink 老师第 45~48 节，接昨天的盒子模型往下走：先把尺寸计算彻底算清——content-box 与 border-box 下 width 到底指什么、全局重置为什么建议带伪元素、100% 宽度为什么会溢出；再把 background 的六个子属性逐个拆开，讲透百分比定位的坐标原理、cover 与 contain 的取舍；然后是背景复合写法的「一行写完也一行清空」和 background-attachment 的固定背景/视差；最后用 linear-gradient 画渐变与条纹，并在文字渐变上把三个必写条件一次说明白。"
---

> 按你的要求，从这一期开始不再生成 Xmind 思维导图，复盘交给文中的表格、结构图和文末自测清单来完成。

![content-box 与 border-box 的尺寸差异](/images/css-box-sizing-background.svg)

---

## 45 · 尺寸计算：box-sizing 到底改了什么

### 一句话理解

> 同一句 `width: 300px`，在 content-box 下说的是「内容区 300」，在 border-box 下说的是「整个盒子 300」。`box-sizing` 回答的就是那个被我们一直含糊掉的问题——**你嘴里说的宽度，指的是哪一段**。

昨天已经知道盒子有四层：content → padding → border → margin。昨天也顺手用过 `box-sizing: border-box` 救场，但没把它单独算清楚。这一节补上，因为它是后面所有布局的「数感」基础。

### 两种盒模型

| 属性值 | 默认？ | `width` / `height` 指什么 | 实际总宽公式 | 谁在用 |
|---|---|---|---|---|
| `content-box` | ✅ 初始值 | 只指**内容区** | `width + padding×2 + border×2` | 浏览器默认、老代码 |
| `border-box` | ❌ 要显式声明 | 指**内容+内边距+边框** | 就是 `width` 本身 | 现代项目几乎全开 |

### 算一笔账（同一个盒子，两种模型）

```css
.box {
  width: 300px;
  padding: 20px;
  border: 4px solid #333;
  margin: 0 10px;
}
```

| 模型 | 内容区宽 | 实际宽度（看得见的边界） | 占位宽度（含 margin） |
|---|---|---|---|
| `content-box` | 300 | 300 + 40 + 8 = **348px** | 348 + 20 = **368px** |
| `border-box` | 300 − 40 − 8 = **252px** | **300px** | 300 + 20 = **320px** |

两个结论先钉死：

1. **`margin` 永远不进 `width`**，`box-sizing` 也管不到它——它长在盒子外面。
2. `border-box` 不是「把盒子变小」，而是**把 padding 和 border 的份额从 width 里扣出去**，总宽才是你写的那个数。

### 为什么一定要开 border-box

1. **设计稿是「含边框」标注的**：标注 200×120 的按钮，你不会想减一遍 padding 再减一遍 border。
2. **百分比布局不再溢出**：`width: 100%` + `padding: 20px` 在 content-box 下必然横向溢出 40px，出现横向滚动条。
3. **改 padding 不再连锁改总宽**：调个内边距，整行布局不会被推歪。
4. **等分列不再算错**：3 个 `width: 33.33%` 各带 padding，content-box 下必定换行。

第 2、4 点是新手最常遇到的「玄学 bug」，其实全是这一句话没设。

### 全局重置（一次写好，终身受益）

```css
/* 推荐写法：继承式 */
html { box-sizing: border-box; }
*, *::before, *::after { box-sizing: inherit; }

/* 等效的粗暴版，也是绝大多数脚手架的做法 */
*, *::before, *::after { box-sizing: border-box; }
```

两个细节值得讲：

- **为什么要带 `::before / ::after`**：`*` 选择器**不匹配伪元素**，只写 `*, {}` 会让伪元素仍然停留在 content-box，做装饰条、遮罩层时宽度算不准。
- **为什么推荐 `inherit`**：以后接第三方组件或富文本区，如果它内部依赖 content-box，只改那个容器一句 `box-sizing: content-box`，内部全部自动回退，不用逐个改。

### 冷知识与边界情况

- **表单控件默认就是 border-box**：浏览器 UA 样式里 `input / textarea / select` 都声明了 `box-sizing: border-box`，所以它们一直「所见即所得」，这也是为什么早年只有 div 会突然变宽。
- **对行内元素基本无效**：`inline` 的 `width / height` 本来就不生效，谈不上从哪儿算起；要控尺寸得先 `inline-block`。
- **width 小于 padding+border 时**：content 会被压到 0，内容溢出盒子之外。`border-box` 只是不算错，不会替你兜住内容。
- **`min-width / max-width / min-height / max-height` 同样受它影响**，不只是 width/height。
- **`flex-basis` 也吃 box-sizing**（border-box 时包含 padding/border）；不过 `flex: 1` 的 basis 是 0，所以最常见的用法不受波及。

### 什么时候仍然该用 content-box

- 需要精确控制「装东西的区域」尺寸时，比如 canvas 容器、某些图表库要求容器内容区为固定像素；
- 想把 padding 当成「额外留白」、让总宽随内容自然生长（较少见）；
- 第三方组件或老项目的既有实现，改动成本远大于收益。

> 判断标准一句话：**你要的是「看得见的盒子多大」→ border-box；你要的是「装东西的地方多大」→ content-box**。日常 95% 是前者。

---

## 46 · background 常见属性：一个盒子能贴几张皮

背景不是一个属性，而是**一组子属性的组合**。先把它们拆开认脸，后面复合写法才不会乱。

### 六个主力子属性

| 子属性 | 干什么 | 常用值 | 初始值 |
|---|---|---|---|
| `background-color` | 打底颜色 | `red` / `#f60` / `rgb()` / `rgba()` / `transparent` | `transparent` |
| `background-image` | 贴图（可多张） | `url(...)` / `linear-gradient(...)` | `none` |
| `background-repeat` | 平铺方式 | `repeat` / `no-repeat` / `repeat-x` / `repeat-y` | `repeat` |
| `background-position` | 图的落点 | 方位名词 / 百分比 / 长度 | `0% 0%` |
| `background-size` | 图的缩放 | `auto` / `cover` / `contain` / 数值 / 百分比 | `auto` |
| `background-attachment` | 跟随谁滚动 | `scroll` / `fixed` | `scroll` |

另外两个加分项，等讲文字渐变时会用到：`background-clip`（背景画到哪一层，初始 `border-box`）、`background-origin`（定位从哪层算起，初始 `padding-box`）。

### background-color

```css
background-color: #f5f5f5;
background-color: rgba(0, 0, 0, .5);   /* 半透明 */
background-color: transparent;          /* 初始值，等于没设 */
```

几个实用点：

- **图的加载占位**：先给 `background-color: #eee`，图没加载出来时是灰底而不是一片白，观感好得多。
- **透明 PNG 的底色**：透明图标底下垫一个色，图标才看得清。
- **层级关系**：`color` 在最底层，`image` 盖在它上面——所以半透明图会透出底色。

### background-image

```css
background-image: url(../images/banner.jpg);
```

- **背景图不撑开盒子**，这是和 `<img>` 最本质的区别：`<img>` 是内容，会占位、会把父级撑高；背景图只是「贴皮」，多大都不影响元素尺寸，只被盒子裁切。
- **路径是相对 CSS 文件**，不是相对 HTML 文件。这是本地能看见、放到服务器就 404 的头号原因。
- **多张图用逗号分隔**，写在**前面**的在**上面**（上层先画）。

### background-repeat

| 值 | 行为 |
|---|---|
| `repeat` | 横竖都铺满（默认） |
| `no-repeat` | 只显示一张 |
| `repeat-x` / `repeat-y` | 只横向 / 只纵向铺 |
| `space` | 完整平铺，缝隙均分（不裁图） |
| `round` | 平铺并缩放，保证不留缝（会轻微变形） |

`space` / `round` 知道有就行，日常 99% 是 `no-repeat`。

### background-position（本节最值得抠的一点）

```css
background-position: left top;                 /* 左上 */
background-position: center;                   /* 一个值 → 第二个默认 center */
background-position: right 20px bottom 10px;   /* 四值写法：先贴边，再偏移 */
```

**百分比到底怎么算**，一句话：

```text
图片左边缘的偏移量 = (容器宽 − 图片宽) × 百分比
```

所以：

| 百分比 | 实际效果 |
|---|---|
| `0%` | 图左边缘贴容器左边缘 |
| `50%` | 图中心对容器中心（= `center`） |
| `100%` | 图右边缘贴容器右边缘 |
| 图比容器大时 | 公式照样成立，只是偏移量为负 |

这就解释了那个经典疑惑：**`background-position: 50%` 和 `center` 一样，但和 `left 50%` 完全不同**——后者是「贴左边缘，再从左边往回推半个容器宽」，图不在中间。

### background-size

| 值 | 行为 | 会变形吗 | 会裁切吗 | 典型场景 |
|---|---|---|---|---|
| `auto` | 用原图尺寸 | 否 | 可能被裁 | 默认 |
| `cover` | 等比放大到**盖满**容器 | 否 | ✅ **裁掉溢出部分** | 全屏 hero、卡片封面 |
| `contain` | 等比缩放到**完整装下** | 否 | ❌ 不裁，但**留白** | 商品图、logo、验证码 |
| `100% 100%` | 强行拉到容器宽高 | ✅ **变形** | 不裁 | 极少用（要的就是拉伸） |
| `50% auto` | 宽一半、高按比例 | 否 | — | 半透明水印 |

一句话记：**`cover` 是「铺满、宁可裁」，`contain` 是「看全、宁可留白」**。判断用哪个，就问自己「这张图的信息完整重要，还是铺满好看重要」。

### 背景和内容谁在上面

绘制顺序是固定的：

```text
background（最底） → 内容（文字 / 图片） → 子元素
```

背景**永远在内容下面，且不能靠 z-index 翻身**——它属于元素自身的绘制层，不是独立图层。想做「压在图上的半透明遮罩」，正确姿势是加伪元素：

```css
.hero {
  position: relative;
  background: url(hero.jpg) center / cover no-repeat;
}
.hero::before {
  content: "";
  position: absolute; inset: 0;
  background: rgba(0, 0, 0, .45);
}
.hero > * { position: relative; }   /* 让文字浮到遮罩之上 */
```

这也是后面做「渐变蒙层」的标准手法：遮罩不是背景的一部分，而是**一个新的、盖在背景上的层**。

---

## 47 · 背景复合写法：一行写完，也一行清空

### 语法

```css
background: [color] [image] [repeat] [attachment] [position / size] [clip] [origin];
```

真正常用的就四段：`颜色  图片  平铺  位置/尺寸`。

```css
background: #f5f5f5 url(bg.jpg) no-repeat center / cover fixed;
```

两条硬规则：

1. **顺序自由，但 `position` 与 `size` 必须连在一起、用斜杠分隔，且 size 在斜杠后面**。写成 `cover center` 会被判为非法，**整条声明被丢弃**（不是只丢一个值）。
2. 只写一个位置值（`... center ...`）= 水平垂直都居中。

### 三个可以直接抄的模板

```css
/* ① 全屏封面（最常见的万能公式） */
.hero { background: #000 url(hero.jpg) center / cover no-repeat fixed; }

/* ② 列表项小图标：靠右 8px，尺寸 16×16 */
.item { background: #fff url(icon.png) 8px center / 16px 16px no-repeat; }

/* ③ 纹理底：小图平铺 */
.page { background: #fafafa url(paper.png) repeat; }
```

### 大坑：复合写法「省略即重置」

`background: red;` **不是「只把颜色改成红」**，而是把八个子属性全部重置成初始值，再设颜色。所以这段代码会出事：

```css
.box  { background: url(a.png) no-repeat center / cover; }
.box:hover { background: red; }        /* ← 图、平铺、尺寸全没了 */
```

正确做法：hover 里只写子属性，或者把完整背景重写一遍。

```css
.box:hover { background-color: red; }  /* ✅ 只改颜色 */
```

被重置成什么样，记住这张表（「没写 = 回到这里」）：

| 子属性 | 被重置为 |
|---|---|
| `background-color` | `transparent` |
| `background-image` | `none` |
| `background-repeat` | `repeat` |
| `background-position` | `0% 0%` |
| `background-size` | `auto` |
| `background-attachment` | `scroll` |
| `background-clip` | `border-box` |
| `background-origin` | `padding-box` |

同理，`background: url(x.png) no-repeat;` 会把尺寸打回 `auto`、位置打回左上——**复合写法要么写全，要么就老老实实写子属性**。

### 多层背景

```css
background-image: url(dots.png), linear-gradient(135deg, #667eea, #764ba2);
background-repeat: no-repeat, no-repeat;
background-position: right 20px bottom 20px, 0 0;
background-size: 80px, cover;
```

复合写法同样可以叠多层，逗号分隔：

```css
background:
  url(dots.png) right 20px bottom 20px / 80px no-repeat,
  linear-gradient(135deg, #667eea, #764ba2);
```

三条规律：

1. **先写的层在上面**（和 `background-image` 的顺序一致）；
2. **只有最后一层能写颜色**，前面几层的 `background-color` 会被忽略；
3. 每层的 `repeat / position / size` 按逗号**一对一对应**，少写会循环取值——层数多时建议分行写，别挤一行。

### background-attachment: fixed —— 固定背景与「视差」

```css
body {
  background: url(sky.jpg) center / cover no-repeat fixed;
}
```

- `scroll`（默认）：背景的坐标系是**元素自己**，元素滚出屏幕，背景跟着一起滚。
- `fixed`：背景的坐标系是**视口**，滚动时图稳稳不动，内容在图上滑过去 → 天然的视差感。

直观感受：给一个 800px 高的区块设 fixed 背景，页面滚动时你看到的是**同一块图片区域**，像隔着一扇固定的窗看风景。

四个注意事项（都是真会踩的）：

1. `fixed` + `background-size: cover` 是按**视口**算尺寸的，元素比视口宽时图会糊、会露底。
2. **iOS Safari 对 `background-attachment: fixed` 支持很差**，经常表现成 `scroll` 或者直接错位。移动端优先改用「`position: fixed` + `z-index: -1` 的独立图层」。
3. `fixed` 背景会触发额外的合成层，长页面里别给几十个元素都挂，性能会掉。
4. 想要「只让这张图钉住、内容滑过」，用定位伪元素反而更可控：

```css
.bg-layer {
  position: fixed; inset: 0; z-index: -1;
  background: url(sky.jpg) center / cover no-repeat;
}
```

---

## 48 · 渐变：background-image 的一种「函数」

### 先纠正一个高频误解

```css
background-color: linear-gradient(90deg, #f00, #00f);  /* ❌ 无效，渐变不是颜色 */
background-image: linear-gradient(90deg, #f00, #00f);  /* ✅ */
background: linear-gradient(90deg, #f00, #00f);        /* ✅ 复合写法，等价 */
```

渐变在 CSS 里属于 **image**（和 `url()` 同级）。所以它能：叠多层、被 `background-size` 拉伸、被 `background-clip` 裁切、被 `border-image` 使用、甚至做遮罩（`mask`）。

### linear-gradient 语法

```css
linear-gradient([方向或角度,] 色标1 [位置1], 色标2 [位置2], ...)
```

角度对照表（背下这张就够了）：

| 写法 | 方向 |
|---|---|
| `to top` / `0deg` | 从下往上 |
| `to right` / `90deg` | 从左往右 |
| `to bottom` / `180deg` | 从上往下（**默认**） |
| `to left` / `270deg` | 从右往左 |
| `45deg` | 指向右上（左下 → 右上） |

⚠️ 角度是**顺时针**、`0deg` **朝上**。所以 `to right` 是 `90deg`，不是 `0deg`；写反了渐变就整个转了个身。

```css
background: linear-gradient(90deg, #ff9a9e, #fad0c4);              /* 左 → 右 */
background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);     /* 经典对角线 */
background: linear-gradient(to bottom, rgba(0,0,0,0), rgba(0,0,0,.6)); /* 底部渐隐 */
```

色标位置：不写就自动均分；`#f00 20%` 表示红在 20% 处；**两个色标位置相同 → 硬边**（下面就用它做条纹）。

### 用硬边做条纹（不需要图）

```css
.stripes {
  background: repeating-linear-gradient(
    45deg,
    #ffffff 0 10px,
    #f1f5f9 10px 20px
  );
}
```

`颜色 0 10px` 是「从 0 到 10px 都用这个颜色」的两点简写，两个色标首尾相接就出现**硬边条纹**，这就是斑马纹、斜纹背景的做法。

### 另外两种渐变（认识就行）

```css
/* 径向：从中心向外散开，做光晕/聚焦 */
background: radial-gradient(circle at center, #ffffff, #e2e8f0);

/* 锥形：绕中心一圈，做色环、饼图、进度环 */
background: conic-gradient(#f00, #ff0, #0f0, #0ff, #00f, #f0f, #f00);
```

### 文字渐变：四行代码，三个必写

```css
.gradient-text {
  background-image: linear-gradient(90deg, #667eea, #f093fb);
  -webkit-background-clip: text;      /* 让背景只画在字形内部 */
          background-clip: text;
  color: transparent;                 /* 字色透明，露出背景 */
  -webkit-text-fill-color: transparent; /* Safari / 旧版 Chrome 兜底 */
}
```

原理一句话：**把背景裁成「文字的形状」，再把文字本身涂透明**。少任何一步都会失败：

| 少写什么 | 现象 |
|---|---|
| `background-clip: text` | 出现一整块渐变方块，字被埋在里面 |
| `color: transparent` | 文字还是正常颜色，完全看不出渐变 |
| `-webkit-` 前缀 | Safari / 旧 Chrome 里变透明块或干脆不生效 |
| 元素没宽度 | 渐变按内容长度算，短标题看着像纯色（加 padding 或 `inline-block` 撑开） |

其他坑：

- **不要给渐变文字加 `text-shadow`**：阴影也会被 clip 掉，看起来像重影。
- 调试时先把 `background-color` 关掉，它同样会被裁进字形，容易以为是渐变没生效。
- 多行文字也能裁，但渐变是**跨行连续**的（整个元素共用一张背景），想让每行独立就得拆成多行元素。
- 字号很小时渐变两端容易「看不出变化」，把色标距离拉近（如 `20% ~ 80%`）会更明显。

### 渐变还能这么玩

```css
/* 渐变边框：两层背景，内层用 padding-box 盖住中间 */
.card {
  border: 2px solid transparent;
  background:
    linear-gradient(#fff, #fff) padding-box,
    linear-gradient(90deg, #667eea, #f093fb) border-box;
}

/* 按钮 hover 的光带扫过：background-size 200% + 位移 */
.btn {
  background-image: linear-gradient(90deg, #6366f1, #a855f7, #6366f1);
  background-size: 200% 100%;
  background-position: 0 0;
  transition: background-position .45s ease;
}
.btn:hover { background-position: 100% 0; }
```

---

## 综合 Demo：视差封面 + 渐变蒙层 + 文字渐变大标题

把今天四节的东西全塞进一屏，每个注释都对应上面的知识点。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>Hero：视差背景 + 文字渐变</title>
<style>
  /* 1. 全局重置：从"看得见的盒子"开始算尺寸 */
  html { box-sizing: border-box; }
  *, *::before, *::after { box-sizing: inherit; }

  body { margin: 0; font-family: system-ui, "Microsoft YaHei", sans-serif; color: #0f172a; }

  /* 2. 视差封面：fixed 让背景相对视口固定，内容滑过时图不动 */
  .hero {
    position: relative;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    padding: 0 8%;
    background: #1e1b4b url(hero.jpg) center / cover no-repeat fixed;
  }

  /* 3. 蒙层：::before 铺满，压在图上、在文字下 */
  .hero::before {
    content: "";
    position: absolute;
    inset: 0;
    background: linear-gradient(160deg,
      rgba(15, 23, 42, .85),
      rgba(76, 29, 149, .65) 60%,
      rgba(15, 23, 42, .90));
  }
  .hero > * { position: relative; }   /* 4. 文字浮到蒙层之上 */

  /* 5. 文字渐变：背景裁进字形 + 文字透明 */
  .title {
    margin: 0 0 18px;
    font-size: clamp(32px, 6vw, 64px);
    font-weight: 800;
    background-image: linear-gradient(90deg, #a5b4fc, #f0abfc 50%, #fca5a5);
    -webkit-background-clip: text;
            background-clip: text;
    color: transparent;
    -webkit-text-fill-color: transparent;
  }
  .subtitle { max-width: 620px; margin: 0; color: rgba(255,255,255,.8); line-height: 1.8; }

  /* 6. 三张等宽卡片：border-box 之后，加 padding 也不溢出 */
  .cards { display: flex; gap: 18px; margin-top: 48px; }
  .card {
    flex: 1;
    padding: 22px 24px;
    border: 1px solid rgba(255,255,255,.18);
    border-radius: 14px;
    background: rgba(255,255,255,.08);
  }
  .card h3 { margin: 0 0 8px; font-size: 18px; color: #fff; }
  .card p  { margin: 0; font-size: 14px; line-height: 1.7; color: rgba(255,255,255,.72); }

  /* 7. 渐变按钮：200% 背景 + 位移做光带 */
  .btn {
    align-self: flex-start;
    margin-top: 40px;
    padding: 14px 30px;
    border: 0;
    border-radius: 999px;
    font-size: 16px;
    font-weight: 600;
    color: #fff;
    cursor: pointer;
    background-image: linear-gradient(90deg, #6366f1, #a855f7, #6366f1);
    background-size: 200% 100%;
    background-position: 0 0;
    transition: background-position .45s ease;
  }
  .btn:hover { background-position: 100% 0; }
</style>
</head>
<body>
  <section class="hero">
    <h1 class="title">把背景玩明白</h1>
    <p class="subtitle">box-sizing 决定盒子多大，background 决定盒子长什么样，gradient 决定它好不好看——三件事叠在一起，就是一屏能用的首屏。</p>

    <div class="cards">
      <div class="card"><h3>尺寸</h3><p>border-box 让 width 就是总宽，改 padding 不再连锁改布局。</p></div>
      <div class="card"><h3>背景</h3><p>center / cover 是全屏封面的万能公式，fixed 换来的是视差。</p></div>
      <div class="card"><h3>渐变</h3><p>渐变属于 image，能裁、能叠、能当文字色，还能动起来。</p></div>
    </div>

    <button class="btn">继续看下一节</button>
  </section>
</body>
</html>
```

这一屏用到了：全局 `border-box` 重置、`center / cover no-repeat fixed` 复合写法、伪元素蒙层 + 层级控制、文字渐变四件套、`flex: 1 + gap` 的三等分卡片、`background-size: 200%` 的渐变按钮。**45~48 节的考点全在这六个技巧里。**

---

## 今日踩坑小结（13 条，都是真会踩的）

1. **开了 border-box，卡片却比别人窄** → padding 从 width 里扣了，内容区变窄。这不是 bug，是「总宽一致」的代价。
2. **只写了 `*, {}` 漏伪元素** → 装饰条、遮罩层宽度算错，补上 `::before / ::after`。
3. **`width: 100%` + padding 出现横向滚动条** → content-box 下必然溢出，换 border-box。
4. **设了 `width: 300px` 内容却溢出来** → padding+border 加起来超过 300，content 被压到 0。
5. **背景图不显示** → 路径相对的是 **CSS 文件**不是 HTML；url 里有空格没加引号也会失效。
6. **以为背景图会把盒子撑大** → 那是 `<img>` 的行为，背景图永远不占位。
7. **`background-position: 50%` 和 `left 50%` 效果不同** → 百分比是「图的 p% 对容器的 p%」，`left 50%` 是从左边缘往回推半个容器。
8. **`background-size: 100% 100%` 图被拉扁** → 想等比就 `cover`，或 `100% auto`。
9. **hover 只想换底色，结果图没了** → 复合写法省略即重置，改成 `background-color`。
10. **多层背景的颜色只生效一层** → `background-color` 只能写在最后一层。
11. **`background: ... cover center` 整条失效** → 位置和尺寸顺序反了，必须 `position / size`。
12. **`background-attachment: fixed` 在手机上错位** → iOS Safari 支持差，改用 `position: fixed` 的独立图层。
13. **文字渐变变成一块透明色** → 少了 `color: transparent` 或 `-webkit-` 前缀，或被 `text-shadow` 干扰。

> 排查顺序：先看 DevTools 里 **Computed 面板**的四层彩色方块（它写的就是盒子模型），再看 **Backgrounds** 有没有被划掉的声明（顺序非法的复合写法会在这里显示删除线），最后用 `outline: 1px solid red` 确认真实边界。

---

## 实战练习（由易到难，40 分钟内能做完）

**① 精确尺寸盒子（10 分钟）**
做三个并排卡片，容器 960px、间距 16px，要求每张卡片**总宽正好等于容器三等分**。先用 content-box 体会溢出，再切 border-box 验证不溢出。验收：DevTools 量三张卡片外宽之和 = 960px。

**② cover 与 contain 对照（15 分钟）**
同一张图，做两个 320×200 的盒子：一个 `center / cover`，一个 `center / contain`。观察 cover 裁掉了哪些部分、contain 留白了多少。然后给 cover 那张加一层 `rgba(0,0,0,.4)` 的伪元素蒙层，把文字压在上面。

**③ 渐变标题卡（15 分钟）**
做一张卡片：渐变边框（`padding-box` + `border-box` 双层背景）、标题用文字渐变、底部一个 `background-size: 200%` 的渐变按钮。验收：hover 时按钮出现光带横扫，标题渐变在 16px 字号下依然有可见色差。

---

## 自测清单

- [ ] 说出 content-box 与 border-box 下 `width` 分别指什么
- [ ] `width: 300 + padding: 20 + border: 4 + margin: 0 10px`，两种模型的总宽、占位宽各是多少
- [ ] 全局重置为什么建议写 `*, *::before, *::after`
- [ ] 为什么推荐 `html { box-sizing: border-box }` + `* { box-sizing: inherit }`
- [ ] 哪些元素默认就是 border-box？为什么
- [ ] `box-sizing` 会改变 margin 吗
- [ ] `width` 小于 padding+border 时会发生什么
- [ ] `background-color` 和 `background-image` 谁在上面
- [ ] 背景图和 `<img>` 在「是否占位」上的本质区别
- [ ] 背景图路径相对谁解析
- [ ] `background-position: 50%` 的计算原理，写出公式
- [ ] `50%`、`center` 与 `left 50%` 三者的差别
- [ ] `cover` 与 `contain` 的取舍标准
- [ ] 给背景加半透明遮罩，为什么不能用 z-index，正确做法是什么
- [ ] 背景复合写法中 `position` 与 `size` 的书写规则
- [ ] `background: red;` 会重置哪些子属性（至少说出五个）
- [ ] 多层背景的层级顺序与颜色的书写限制
- [ ] `background-attachment: fixed` 的坐标系是谁，移动端为什么慎用
- [ ] 渐变属于 color 还是 image，能用在哪些属性上
- [ ] `0deg` 指向哪里？`to right` 等于多少度
- [ ] 用 `repeating-linear-gradient` 写出 45° 斑马纹
- [ ] 文字渐变的三个必写条件，各漏一个会怎样
- [ ] 渐变文字为什么不能加 `text-shadow`

---

## 附：今日结构图与复盘方式

- 结构图：[content-box / border-box 尺寸对照](/images/css-box-sizing-background.svg)

按你的要求，本期起**不再生成 Xmind 图谱**，复盘就靠三样东西：上面那张结构图确认尺寸关系 → 文中表格对齐「属性—行为—坑」→ 自测清单逐条口述。能全是就过，卡住的那条直接回去点开 DevTools 验一遍，比看图谱快。

> 盒子模型到这里基本闭环：**尺寸怎么算、皮怎么贴、渐变怎么画**。下一篇进浮动与定位，盒子就要开始「脱离文档流」了 💪
