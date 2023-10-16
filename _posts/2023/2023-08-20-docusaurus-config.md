---
title: docusaurus.config.js  的配置项
date: 2023-08-20 14:31:57 +0800
categories: [文章]
tags: [docusaurus, reactjs]
---


:::tips
检查 docusaurus.config.js  参考以获取详尽的选项列表。
:::
Docusaurus 对配置有独特的看法。我们鼓励您将有关您网站的信息集中到一处。我们保护该文件的字段，并促进在您的站点上访问该数据对象。
保持良好维护的 docusaurus.config.js 可以帮助您、您的协作者和开源贡献者能够专注于文档，同时仍然能够自定义站点。

### 声明 `docusaurus.config.js` 的语法
`docusaurus.config.js` 文件在 Node.js 中运行，并应导出：

- 一个配置对象（a config object）
- 创建配置对象的函数（a function that creates the config object）
:::info
docusaurus.config.js 文件仅支持 [CommonJS](https://flaviocopes.com/commonjs/) 模块系统：

- 必填（Required）：使用 `module.exports = /* your config*/` 导出 Docusaurus 配置
- 自选（Optional）：使用 `require('lib')` 导入 Node.js 包
- 自选（Optional）：在异步函数中使用 `await import('lib')`（动态导入）来导入仅限 ESM 的Node.js 包
:::
 
Node.js 使我们能够以各种等效方式声明 Docusaurus 配置，并且以下所有配置示例都会产生完全相同的结果：
```javascript
module.exports = {
  title: 'Docusaurus',
  url: 'https://docusaurus.io',
  // your site config ...
};
```
```javascript
const config = {
  title: 'Docusaurus',
  url: 'https://docusaurus.io',
  // your site config ...
};

module.exports = config;
```
```javascript
module.exports = function configCreator() {
  return {
    title: 'Docusaurus',
    url: 'https://docusaurus.io',
    // your site config ...
  };
};
```
```javascript
module.exports = async function createConfigAsync() {
  return {
    title: 'Docusaurus',
    url: 'https://docusaurus.io',
    // your site config ...
  };
};
```
:::success
**使用仅 ESM 的软件包：**
使用异步配置创建器对于导入仅限 ESM 的模块（尤其是大多数 Remark 插件）非常有用。由于动态导入，可以导入此类模块：
:::
```javascript
module.exports = async function createConfigAsync() {
  // Use a dynamic import instead of require('esm-lib')
  const lib = await import('lib');

  return {
    title: 'Docusaurus',
    url: 'https://docusaurus.io',
    // rest of your site config...
  };
};
```
### docusaurus.config.js 包含的内容
即使您正在开发网站，您也不应该从头开始编写 `docusaurus.config.js`。所有模板都附带 `docusaurus.config.js`，其中包含常用选项的默认值。
但是，如果您对配置的设计和实现方式有深入的了解，这会很有帮助。
Docusaurus 配置的高级概述可分为：

- 站点元数据 site metadata
- 部署配置 Deployment configurations
- 主题、插件和预设配置 Theme, plugin, and preset configurations
- 自定义配置 Custom configurations
#### site metadata 站点元数据
站点元数据包含基本的全局元数据，例如标题 title、网址 url、基础路径 baseUrl 和 图标 favicon。
它们可用于多个地方，例如网站的标题和标题、浏览器选项卡图标、社交共享（Facebook、Twitter）信息，甚至生成提供静态文件的正确路径。
#### 部署配置
当您使用部署 deploy 命令部署站点时，将使用部署配置，例如 `projectName`、`organizationName` 和可选的`deploymentBranch`。
建议查看部署文档（deployment docs）以获取更多信息。
#### 主题、插件和预设配置 Theme, plugin, and preset configurations
分别在主题[ theme](https://www.docusaurus.cn/docs/using-plugins#using-themes)、插件[ plugin ](https://www.docusaurus.cn/docs/using-plugins)和[预设字段 preset configurations ](https://www.docusaurus.cn/docs/using-plugins#using-presets)中列出您网站的主题、插件和预设。这些通常是 npm 包：
```javascript
module.exports = {
  // ...
  plugins: [
    '@docusaurus/plugin-content-blog',
    '@docusaurus/plugin-content-pages',
  ],
  themes: ['@docusaurus/theme-classic'],
};
```
