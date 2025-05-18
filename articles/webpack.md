
~~# 爬虫知识笔记：Webpack 与 JS 逆向

## 什么是 Webpack

Webpack 是前端模块打包工具，它可以将多个模块（如 HTML、CSS、JavaScript 等）进行压缩、合并与优化，最终打包成浏览器可识别并执行的静态资源文件。

Webpack 打包后的结构常见如下：

```js
// 模块为数组形式
!function(modules) {
  // 加载器逻辑
}([
  function(module, exports) { /* 模块1 */ },
  function(module, exports) { /* 模块2 */ }
]);

// 模块为对象形式
!function(modules) {
  // 加载器逻辑
}({
  "k1": function(...) { /* 模块1 */ },
  "k2": function(...) { /* 模块2 */ }
});
```

---

## Webpack 与加载器（Loader）的关系

Webpack 可以理解为中央打包工厂，而其中的 Loader（加载器）就是各类“工人”。Webpack 将各类文件传入 Loader，经过处理后统一输出成打包后的模块。

---

## 爬虫与 Webpack 的关系

在 JS 逆向过程中，爬虫常会遇到 Webpack 打包后的 JS 文件。Webpack 带来的挑战包括：

### 1. 动态渲染

Webpack 输出的 JS 页面依赖浏览器动态执行，传统的 `requests` 抓取方式无法获取最终渲染结果。

**解决方案**：使用 Selenium、Playwright 等浏览器自动化工具，等待 JS 执行完成后再提取数据。

### 2. 模块打包与混淆

Webpack 会将大量 JS 模块混淆压缩，变量名通常为 `e`, `t`, `n` 等匿名参数，使前端逻辑变得难以理解。

### 3. 动态定位符生成

配合 Loader，Webpack 会动态生成 `class` 或 `id`，使静态页面结构无法稳定匹配。

**应对策略**：需结合页面行为观察、脚本分析，寻找稳定标志或使用图像识别类手段。

### 4. 隐匿关键逻辑

关键参数或请求过程常被封装在模块函数中，例如 `r("abc123")` 即调用模块。想要找到数据来源，需要从加载器和模块依赖入手。

---

## 如何处理 Webpack 打包的页面

总结为五步操作：

### 步骤一：判断页面是否使用 Webpack

观察源码中是否有 `webpackJsonp`、`__webpack_require__`、或模块结构如 `!function(modules){}` 等关键词。

---

### 步骤二：断点定位模块加载器

在浏览器 DevTools 中断点调试 `r("xxx")` 或自执行函数，寻找加载器入口，通常刷新页面时就能触发。

---

### 步骤三：判断是单文件还是多文件模式

- **单文件 Webpack**：模块都打包在一个文件中。
- **多文件 Webpack**：模块通过多个文件异步加载（如通过 `webpackChunk` 动态插入）。

判断依据：

```js
// 单文件结构
(function(modules) { ... })([ /* 所有模块 */ ]);

// 多文件结构
window.webpackChunk = window.webpackChunk || [];
window.webpackChunk.push([[id], { "moduleId": fn }, ...]);
```

> 多文件结构中，打包文件本体可能为空，真正模块由其他 JS 动态引入。

---

### 步骤四：复制加载器到本地进行分析

1. 替换浏览器环境为 Node 环境（如加上 `window = global`）。
2. 添加 `console.log` 输出模块执行内容。
3. 注释初始化函数，防止页面自动执行。

---

### 步骤五：手动测试模块调用

逐个测试模块是否能被调用，找出含有请求参数、加密逻辑等关键模块。

---

## 小结

Webpack 是当前主流前端构建工具，它对爬虫逆向工程带来了很大挑战。理解其打包逻辑、加载器结构、模块系统等，是进行现代 JS 逆向的必要基础。

推荐实战搭配网站如：

- 某电商类平台
- 滑块验证码类平台
- 字体加密页面