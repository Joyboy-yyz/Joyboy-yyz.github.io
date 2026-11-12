---
title: "2026-11-12 学习日志：HTML5 表单与语义化布局 —— form、input、label、字符实体与科技日报案例"
date: 2026-11-12T12:00:00+08:00
draft: false
tags: ["HTML", "前端", "表单", "form", "input", "label", "语义化", "字符实体", "学习日志"]
categories: ["学习笔记"]
summary: "Pink 老师前端课第 16~21 节：form 表单容器与 text/password 输入框、input 的 radio/checkbox/file/submit 等 type、textarea 文本域、select 下拉框、button、label 辅助标签、HTML 字符实体；并在 B 站补了一遍 HTML5 多媒体（img/audio/video）、表格、语义布局（header/nav/main/aside/footer）的实例，最后用「科技日报」综合案例把整页结构串起来。"
---

## 16 · 表单容器 `<form>` 与文本框 / 密码框

网页上让用户"填东西"的区域都叫**表单**，外层用 `<form>` 包起来。它本身在页面上看不见，负责：

- 收集里面所有控件的数据；
- 指定数据要发到哪个 URL（`action`）、用什么方法发（`method="get/post"`）。

```html
<form action="/login" method="post">
  <!-- 单行文本框 -->
  <input type="text" name="username" placeholder="请输入用户名">

  <!-- 密码框：输入内容自动变成圆点 -->
  <input type="password" name="pwd" placeholder="请输入密码">
</form>
```

### 常用属性（input 全家通用）

| 属性 | 作用 |
|---|---|
| `type` | 控件类型：text / password / radio / checkbox / file / submit / button … |
| `name` | **提交数据时的字段名**，后端靠它取值，不写就发不出去 |
| `value` | 控件的默认值 / 实际提交的值 |
| `placeholder` | 灰色提示文字，用户一打字就消失 |
| `disabled` | 禁用，灰掉且不能点，数据也不会提交 |
| `readonly` | 只读，能看不能改，**数据会提交** |
| `required` | 必填，空着不让提交（HTML5 自带校验） |

> 大白话：`name` 是"这个格子在表格里叫什么列名"，`value` 是"格子里现在填了什么"。后端收到的就是 `name=value` 这种键值对。

---

## 17 · input 家族：单选、复选、文件域

### 单选框 `radio`（同一组只能选一个）

```html
<!-- name 必须一样，浏览器才会把它们当成"一组互斥" -->
<p>性别：
  <input type="radio" name="gender" value="male" id="male">
  <label for="male">男</label>

  <input type="radio" name="gender" value="female" id="female" checked>
  <label for="female">女</label>
</p>
```

要点：
- 同组 radio 的 `name` **必须相同**，否则可以多选；
- `checked` 属性表示"默认选中"；
- 一定要配 `<label>`，否则点文字没反应，体验很烂（下面 19 节细讲）。

### 复选框 `checkbox`（可多选）

```html
<p>爱好：
  <input type="checkbox" name="hobby" value="read" checked> 阅读
  <input type="checkbox" name="hobby" value="code"> 写代码
  <input type="checkbox" name="hobby" value="game"> 打游戏
</p>
```

> 注意：复选框的 `name` 通常写成 `hobby`（后端收到的是数组），而不是 `hobby1 / hobby2`。

### 文件域 `file`

```html
<input type="file" name="avatar">
<!-- 多选：加 multiple -->
<input type="file" name="photos" multiple>
```

### 提交按钮 / 普通按钮

```html
<input type="submit" value="登录">   <!-- 提交整个 form -->
<input type="reset"  value="重置">   <!-- 清空表单 -->
<input type="button" value="点我">   <!-- 普通按钮，配合 JS -->
```

### HTML5 新增的好用 type

```html
<input type="email"    name="email">     <!-- 自动校验邮箱格式 -->
<input type="number"   name="age" min="0" max="120">
<input type="date"     name="birthday">  <!-- 手机端直接弹日历 -->
<input type="color"    name="favcolor">  <!-- 调色板 -->
<input type="range"    name="volume" min="0" max="100">
```

---

## 18 · 文本域 `<textarea>`、下拉表单 `<select>` 与 `<button>`

### 多行文本域

```html
<textarea name="msg" rows="5" cols="30" placeholder="请留言……"></textarea>
```

- 和 `<input type="text">` 的区别：能换行、能写大段文字；
- `rows` 行数、`cols` 列数只是初始大小，用户可以拖拽右下角拉大（想禁止就 CSS 加 `resize: none;`）。

### 下拉菜单

```html
<select name="city">
  <option value="">-- 请选择城市 --</option>
  <optgroup label="湖北">
    <option value="wuhan" selected>武汉</option>
    <option value="yichang">宜昌</option>
  </optgroup>
  <optgroup label="江西">
    <option value="shangrao">上饶</option>
  </optgroup>
</select>
```

- `<option>` 才是选项，`value` 是真正提交的值，标签里的文字是给用户看的；
- `selected` 默认选中；
- 加 `multiple` 可以变成列表框多选。

### 按钮 `<button>`

```html
<button type="submit">提交</button>
<button type="reset">重置</button>
<button type="button" onclick="alert('hi')">普通按钮</button>
```

- 比 `<input type="submit">` 好用：标签中间能塞图片、`<span>`、样式；
- **不写 type 的话，在 form 里默认是 submit**，点一下就提交表单了，常见 bug！

---

## 19 · 辅助标签 `<label>` 与字符实体

### `<label>`：让文字也能点

```html
<!-- 写法一：for 绑定 input 的 id -->
<label for="username">用户名：</label>
<input type="text" id="username" name="username">

<!-- 写法二：直接把 input 包进去 -->
<label><input type="checkbox" name="agree"> 我已阅读并同意用户协议</label>
```

好处：
- 点文字也能聚焦输入框 / 选中单选框，**移动端体验提升巨大**；
- 屏幕阅读器读屏时会念出 label 文字，无障碍（a11y）必备。

### HTML 字符实体（转义字符）

HTML 里有些字符有特殊含义（比如 `<` 是标签开头），想直接显示出来就得用"实体"代替：

| 想要显示 | 实体写法 | 备注 |
|---|---|---|
| `<` | `&lt;` | less than |
| `>` | `&gt;` | greater than |
| 空格 | `&nbsp;` | 不换行空格，比普通空格更"硬" |
| `&` | `&amp;` | |
| © | `&copy;` | 版权符号 |
| ® | `&reg;` | 注册商标 |
| ¥ | `&yen;` | |
| `"` | `&quot;` | 双引号 |

> 最容易踩的坑：在网页里想写一段 `<div>` 代码给人看，直接写浏览器会以为是真标签。必须写成 `&lt;div&gt;`。

---

## 20~21 · 科技日报综合案例：整页结构拆解

Pink 老师带做了一个"科技日报"首页案例，把前 21 节的东西全串起来。页面骨架是这样的：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>科技日报 - 首页</title>
</head>
<body>

  <!-- 1. 顶部导航 -->
  <header>
    <div class="logo">🚀 科技日报</div>
    <nav>
      <a href="#">首页</a>
      <a href="#">国产芯片</a>
      <a href="#">人工智能</a>
      <a href="#">航天探索</a>
    </nav>
  </header>

  <!-- 2. 主体区域：左边主内容 + 右边边栏 -->
  <main>
    <section class="news-list">
      <article>
        <h2>国产 7nm 光刻技术取得新突破</h2>
        <p>2026-11-12 · 记者 振灿</p>
        <p>……正文摘要……</p>
      </article>
    </section>

    <!-- 边栏：热点排行 / 订阅表单 -->
    <aside>
      <h3>订阅我们</h3>
      <form action="/subscribe" method="post">
        <input type="email" name="email" placeholder="输入邮箱" required>
        <button type="submit">订阅</button>
      </form>
    </aside>
  </main>

  <!-- 3. 底部 -->
  <footer>
    <p>© 2026 科技日报 · 好好学习，天天向上</p>
  </footer>

</body>
</html>
```

这里用到的**语义化标签**（HTML5 的灵魂）：

| 标签 | 放什么 | 大白话 |
|---|---|---|
| `<header>` | 页面顶部 / 文章头部 | "这块是头" |
| `<nav>` | 导航链接组 | "这块是菜单" |
| `<main>` | 页面唯一主体内容 | "这是正文" |
| `<section>` | 一个独立章节 | "这是一大节" |
| `<article>` | 一篇独立文章 / 帖子 | "这条新闻" |
| `<aside>` | 侧边栏、广告、相关推荐 | "旁边那栏" |
| `<footer>` | 页脚 / 文章末尾 | "这块是底" |

> 它们视觉上和 `<div>` 一模一样，但**语义完全不同**：SEO 爬虫和读屏软件靠它们理解页面结构。写网页先想"这是头/是正文/是边栏"，再选标签，而不是一股脑全写 `<div>`。

---

## 附：一个 demo 把今天学的全串起来

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>HTML5 标签大全 Demo</title>
</head>
<body>

  <header>
    <h1>🎧 我的 HTML5 练习页</h1>
    <nav>
      <a href="#media">多媒体</a> |
      <a href="#table">表格</a> |
      <a href="#form">表单</a>
    </nav>
  </header>

  <!-- 多媒体：图片 / 音频 / 视频 -->
  <section id="media">
    <h2>多媒体标签</h2>

    <img src="https://picsum.photos/600/300" alt="随机图片" width="600">

    <!-- controls 必须加，否则播放器都不显示 -->
    <audio src="demo.mp3" controls></audio>

    <video src="demo.mp4" controls width="600"
           poster="cover.jpg">
      你的浏览器不支持 video 标签
    </video>
  </section>

  <!-- 表格 -->
  <section id="table">
    <h2>课程表</h2>
    <table border="1">
      <caption>我本周的前端学习安排</caption>
      <thead>
        <tr><th>时间</th><th>内容</th></tr>
      </thead>
      <tbody>
        <tr><td>周一</td><td>HTML 表单</td></tr>
        <tr><td>周三</td><td>CSS 选择器</td></tr>
        <tr><td>周五</td><td>Flex 布局</td></tr>
      </tbody>
    </table>
  </section>

  <!-- 表单：注册小例子 -->
  <section id="form">
    <h2>注册一个账号</h2>
    <form action="/reg" method="post">
      <p>
        <label for="user">用户名：</label>
        <input type="text" id="user" name="username" required>
      </p>

      <p>
        <label for="pwd">密码：</label>
        <input type="password" id="pwd" name="pwd" minlength="6">
      </p>

      <p>性别：
        <label><input type="radio" name="gender" value="m" checked> 男</label>
        <label><input type="radio" name="gender" value="f"> 女</label>
      </p>

      <p>爱好：
        <label><input type="checkbox" name="hobby" value="code"> 写代码</label>
        <label><input type="checkbox" name="hobby" value="music"> 听歌</label>
      </p>

      <p>
        <label for="city">城市：</label>
        <select id="city" name="city">
          <option value="">-- 请选择 --</option>
          <option value="wh">武汉</option>
          <option value="sr">上饶</option>
        </select>
      </p>

      <p>
        <label for="intro">个人简介：</label><br>
        <textarea id="intro" name="intro" rows="3" placeholder="一句话介绍自己"></textarea>
      </p>

      <p>
        <label><input type="checkbox" name="agree" required>
        我同意 <a href="#">用户协议</a></label>
      </p>

      <button type="submit">立即注册</button>
      <button type="reset">清空</button>
    </form>
  </section>

  <aside>
    <h3>今日金句</h3>
    <p>Talk is cheap. Show me the code. &nbsp;——&nbsp; Linus Torvalds</p>
  </aside>

  <footer>
    <p>© 2026 振灿的前端学习笔记 · 每天进步一点点</p>
  </footer>

</body>
</html>
```

---

## 今日踩坑小结

1. **radio 必须同 name 才能互斥**，忘了 name 就变成"能多选的性别框"。
2. **button 不写 type 默认是 submit**，放在 form 里点一下就把表单提交了，想当普通按钮一定要 `type="button"`。
3. **input 不加 name，后端收不到值**；value 才是真正发出去的数据。
4. **label 的 for 要和 input 的 id 一模一样**，点文字才能聚焦。
5. **写 `<` `>` 给人看时要转义成 `&lt;` `&gt;`**，否则浏览器会当真标签解析。
6. 语义标签先想好"这块是头还是尾还是边栏"再写，不要全是 `<div>` 套娃。
7. Hugo 博客文章的 `date` 一定要写当天**已经过去的时间**，不然 Actions 绿了文章也会 404。

明天开始进 CSS 选择器，加油 💪
