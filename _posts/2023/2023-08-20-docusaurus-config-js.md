---
title: docusaurus 入门配置
date: 2023-08-20 14:31:57 +0800
categories: [博客]
tags: [docusaurus, reactjs] 
---


有关示例，请参阅[入门配置](https://www.docusaurus.cn/docs/configuration)。

### 概述
`docusaurus.config.js` 包含您网站的配置，并放置在您网站的根目录中。

该文件使用[ CommonJS ](https://flaviocopes.com/commonjs/)模块系统在 Node.js 中运行，并且应导出站点配置对象或创建它的函数。

例子：
```javascript
module.exports = {
  title: 'Docusaurus',
  url: 'https://docusaurus.io',
  // your site config ...
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
:::info
请参阅声明 `docusaurus.config.js` 的[语法](https://www.docusaurus.cn/docs/configuration#syntax-to-declare-docusaurus-config)以获取更详尽的示例和说明列表。
:::
### 必填字段
#### `titile`

- 类型：string

您网站的标题。将在元数据中使用并用作浏览器选项卡标题。
```javascript
module.exports = {
  title: 'Docusaurus',
};
```
#### `url`

- 类型：string

您网站的 URL。这也可以被视为顶级主机名。例如， `https://facebook.github.io` 是 `https://facebook.github.io/metro/` 的 URL，`https://docusaurus.io` 是 `https://docusaurus.io` 的 URL。该字段与** baseUrl **字段相关。
```javascript
module.exports = {
  url: 'https://docusaurus.io',
};
```
#### `baseUrl`

- 类型：string

您网站的基本 URL。可以认为是宿主之后的路径。 例如，`/metro/` 是 `https://facebook.github.io/metro/` 的基本 URL。对于没有路径的 URL，baseUrl 应设置为 `/`。该字段与** url **字段相关。 始终有前导斜杠 / 和尾随斜杠 /。
```javascript
module.exports = {
  baseUrl: '/',
};
```

### 先填字段
#### `favicon`

- 类型：`string | undefined`

您网站图标的路径；必须是可在链接的 href 中使用的 URL。例如，如果您的网站图标位于`static/img/favicon.ico`
```javascript
module.exports = {
  favicon: '/img/favicon.ico',
};
```
#### `trailingSlash`

- 类型：`boolean | undefined`


允许自定义 URLs 或 链接末尾是否存在尾部斜杠 `/`，以及如何生成静态 HTML 文件：

- `undefined`（默认）：保持 URLs 不变，将 `/docs/myDoc.md` 转换成 `/docs/myDoc/index.html`
- `true`：在 URLs或链接 中添加尾部斜杠，将 `/docs/myDoc.md` 转换成 `/docs/myDoc.md`
- `false`：从 URL/链接中删除尾部斜杠，将 `/docs/myDoc.md` 转换成 `/docs/myDoc.html`
:::info
 每个静态托管提供商以不同的方式提供静态文件（这种行为甚至可能随着时间的推移而改变）
请参阅[部署指南](https://www.docusaurus.cn/docs/deployment)和 [slorber/trailing-slash-guide](https://github.com/slorber/trailing-slash-guide) 选择适当的设置。
:::
#### `i18n`

- 类型：Object

用于[本地化您的站点](https://www.docusaurus.cn/docs/i18n/introduction)的 i18n 配置对象
例子：
```javascript
module.exports = {
  i18n: {
    defaultLocale: 'en',
    locales: ['en', 'fa'],
    path: 'i18n',
    localeConfigs: {
      en: {
        label: 'English',
        direction: 'ltr',
        htmlLang: 'en-US',
        calendar: 'gregory',
        path: 'en',
      },
      fa: {
        label: 'فارسی',
        direction: 'rtl',
        htmlLang: 'fa-IR',
        calendar: 'persian',
        path: 'fa',
      },
    },
  },
};
```

- `defaultLocale` : (1) 基本 URL 中没有其名称的区域设置 (2) 通过 docusaurus start 启动，不带 --locale 选项 (3) 将用于链接 hrefLang='x-default' 标记
- `locales`：您站点上部署的区域设置列表。必须包含defaultLocale。
- `path`：所有语言环境文件夹都相对于的根文件夹。可以是绝对的或相对于配置文件的。默认为 i18n。
- `localeConfigs`：每个区域的单独选项
   - `label`：在区域设置下拉列表中为此区域设置显示的标签。
   - `direction`：ltr（默认）或 rtl（对于[从右到左的语言（right-to-left languages）](https://developer.mozilla.org/en-US/docs/Glossary/rtl)，如波斯语、阿拉伯语、希伯来语等）。用于选择区域设置的 CSS 和 HTML 元属性。
   - `htmlLang`在 `<html lang='...'>`（或任何其他 DOM 标记名称）和`<link ... hreflang='...'>`中使用的 BCP 47 语言标记
   - `calendar`： 用于计算日期纪元的[日历](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/Locale/calendar)。请注意，它不控制实际显示的字符串：`MM/DD/YYYY` 和 `DD/MM/YYYY` 都是 gregory。要选择格式（`DD/MM/YYYY` 或 `MM/DD/YYYY`），请将区域设置名称设置为 `en-GB` 或 `en-US`（`en` 表示 `en-US`）。
   - `path`：该语言环境的所有插件本地化文件夹都相对于的根文件夹。将针对 `i18n.path` 进行解析。默认为区域设置的名称。注意：这对区域设置 `baseUrl` 没有影响 - 基本 URL 的自定义正在进行中。
#### `noIndex`

- 类型：`boolean`

此选项将 `<meta name='robots' content='noindex, nofollow'>` 添加到每个页面，以告诉搜索引擎避免对您的网站建立索引（更多信息请参见此处）。
例子：
```javascript
module.exports = {
  noIndex: true, // Defaults to `false`
};
```
#### `onBrokenLinks`

- 类型：`'ignore' | 'log' | 'warn' | 'throw'`

Docusaurus 检测到任何损坏的链接时的行为。
默认情况下，它会抛出错误，以确保您永远不会发送任何损坏的链接，但如果需要，您可以降低此安全性。
:::tips
 损坏的链接检测仅适用于生产版本（docusaurus 版本）。
:::
#### `onBrokenMarkdownLinks`

- 类型：`'ignore' | 'log' | 'warn' | 'throw'`

Docusaurus 检测到任何损坏的 Markdown 链接时的行为。
默认情况下，它会打印一条警告，让您知道损坏的 Markdown 链接，但您可以根据需要更改此安全性。
#### `onDulicateRoutes`

- 类型：`'ignore' | 'log' | 'warn' | 'throw'`

Docusaurus 检测到任何[重复路由（Duplicate Routes）](https://www.docusaurus.cn/docs/creating-pages#duplicate-routes)时的行为。
默认情况下，它会在运行 `yarn start` 或 `yarn build`后显示警告。
#### `tagline`

- 类型：`string`

您网站的标语。
例子：
```javascript
module.exports = {
  tagline:
    'Docusaurus makes it easy to maintain Open Source documentation websites.',
};
```
#### `organizationName`

- 类型：`string`

拥有存储库的 GitHub 用户或组织。如果您不使用 docusaurus 部署命令，则不需要此命令。
例子：
```javascript
module.exports = {
  // Docusaurus' organization is facebook
  organizationName: 'facebook',
};
```
#### `projectName`

- 类型：`string`

GitHub 存储库的名称。如果您不使用 docusaurus 部署命令，则不需要此命令。
例子：
```javascript
module.exports = {
  projectName: 'docusaurus',
};
```
#### `deploymentBranch`

- 类型：`string`

 将静态文件部署到的分支的名称。如果您不使用 docusaurus 部署命令，则不需要此命令。
例子：
```javascript
module.exports = {
  deploymentBranch: 'gh-pages',
};
```
#### `githubHost`

- 类型：`string`

您的服务器的主机名。如果您使用 GitHub Enterprise，则很有用。如果您不使用 docusaurus 部署命令 `docusaurus deploy`，则不需要此命令。
例子：
```javascript
module.exports = {
  githubHost: 'github.com',
};
```
#### `githubPort`

- 类型：`string`

你的服务器的端口。如果您使用 GitHub Enterprise，则很有用。如果您不使用 docusaurus 部署命令 `docusaurus deploy`，则不需要此命令。
例子：
```javascript
module.exports = {
  githubPort: '22',
};
```
#### `themeConfig`

- 类型：`Object`

用于自定义站点 UI（如导航栏和页脚）的[主题配置（theme configuration）](https://www.docusaurus.cn/docs/api/themes/configuration)对象。
例子：
```javascript
module.exports = {
  themeConfig: {
    docs: {
      sidebar: {
        hideable: false,
        autoCollapseCategories: false,
      },
    },
    colorMode: {
      defaultMode: 'light',
      disableSwitch: false,
      respectPrefersColorScheme: true,
    },
    navbar: {
      title: 'Site Title',
      logo: {
        alt: 'Site Logo',
        src: 'img/logo.svg',
        width: 32,
        height: 32,
      },
      items: [
        {
          to: 'docs/docusaurus.config.js',
          activeBasePath: 'docs',
          label: 'docusaurus.config.js',
          position: 'left',
        },
        // ... other links
      ],
    },
    footer: {
      style: 'dark',
      links: [
        {
          title: 'Docs',
          items: [
            {
              label: 'Docs',
              to: 'docs/doc1',
            },
          ],
        },
        // ... other links
      ],
      logo: {
        alt: 'Meta Open Source Logo',
        src: 'img/meta_oss_logo.png',
        href: 'https://opensource.fb.com',
        width: 160,
        height: 51,
      },
      copyright: `Copyright © ${new Date().getFullYear()} Facebook, Inc.`, // You can also put own HTML here
    },
  },
};
```
#### `plugins`

- 类型：`PluginConfig[]`
```javascript
type PluginConfig = string | [string, any] | PluginModule | [PluginModule, any];
```
有关 PluginModule 的形状，请参阅[插件方法（plugin method references）](https://www.docusaurus.cn/docs/api/plugin-methods)参考。
```javascript
module.exports = {
  plugins: [
    'docusaurus-plugin-awesome',
    ['docusuarus-plugin-confetti', {fancy: false}],
    () => ({
      postBuild() {
        console.log('Build finished');
      },
    }),
  ],
};
```
#### `themes`

- 类型：`PluginConfig[]`

例子：
```javascript
module.exports = {
  themes: ['@docusaurus/theme-classic'],
};
```
#### `presets`

- 类型：`PresetConfig[]`
- 例子：
```javascript
type PresetConfig = string | [string, any];
```
```javascript
module.exports = {
  presets: [],
};
```
#### `markdown`

- 类型：`MarkdownConfig`
```javascript
type MarkdownPreprocessor = (args: {
  filePath: string;
  fileContent: string;
}) => string;

type MDX1CompatOptions =
  | boolean
  | {
    	comments: boolean;
			admonitions: boolean;
			headingIds: boolean;
		};

type MarkdownConfig = {
  format: 'mdx' | 'md' | 'detect';
	mermaid: boolean;
	preprocessor?: MarkdownPreprocessor;
	mdx1Compat: MDX1CompatOptions;
};
```
全局 Docusaurus Markdown 配置。
例子：
```javascript
module.exports = {
  markdown: {
    format: 'mdx',
    mermaid: true,
    preprocessor: ({filePath, fileContent}) => {
      return fileContent.replaceAll('{{MY_VAR}}', 'MY_VALUE');
    },
    mdx1Compat: {
      comments: true,
      admonitions: true,
      headingIds: true,
    },
  },
};
```
#### `customFields`

- 类型：`Object`

Docusaurus 保护 docusaurus.config.js 免受未知字段的影响。要添加自定义字段，请在 customFields 上定义它。
```javascript
module.exports = {
  customFields: {
    admin: 'endi',
    superman: 'lol',
  },
};
```
尝试在配置中添加未知字段将导致构建期间出现错误：
```javascript
Error: The field(s) 'foo', 'bar' are not recognized in docusaurus.config.js
```
#### `staticDirectories`

- 类型：`string[]`

路径数组，相对于站点目录或绝对路径。这些路径下的文件将按原样复制到构建输出。
例子：
```javascript
module.exports = {
  staticDirectories: ['static'],
};
```
#### `headTags`

- 类型：`{ tagName: string; attributes: Object; }[]`

向 HTML `<head>` 中插入标签数组。 这些值必须是包含 `tagName`和 `attributes`两个属性对象：

- `tagName` 必须是确定正在创建的标签的字符串，如 `<link>`。
- `attributes` 必须是属性-值（attribute-value/key-value）映射。

例子：
```javascript
module.exports = {
  headTags: [
    {
      tagName: 'link',
      attributes: {
        rel: 'icon',
        href: '/img/docusaurus.png',
      },
    },
  ],
};
```
#### `scripts`

- 类型：`(string | object)[]`

要加载的脚本数组。 这些值可以是字符串或属性值映射的普通对象。脚本（`<script>`）标签将被插入到 HTML 头部`<head>`。 如果您使用普通对象，则唯一必需的属性是 src，并且允许任何其他属性（每个属性都应该具有布尔/字符串值）
请注意，此处添加的脚本是渲染阻塞的（render-blocking），因此您可能需要向对象添加 `async: true/defer: true` 。
例子：
```javascript
module.exports = {
  scripts: [
    // String format.
    'https://docusaurus.io/script.js',
    // Object format.
    {
      src: 'https://cdnjs.cloudflare.com/ajax/libs/clipboard.js/2.0.0/clipboard.min.js',
      async: true,
    },
  ],
};
```
#### `stylesheets`

- 类型：`(string | object)[]`

需要加载的 CSS 源数组。这些值可以是字符串或属性值映射的普通对象。链接（`<link>`）标签将被插入到 HTML `<head>` 中。
例子：
```javascript
module.exports = {
  stylesheets: [
    // String format.
    'https://docusaurus.io/style.css',
    // Object format.
    {
      href: 'http://mydomain.com/style.css',
    },
  ],
};
```
:::info
默认情况下，`<link>` 标记将具有 `rel='stylesheet'`，但您可以显式添加自定义 `rel` 值来注入任何类型的 `<link>`标记，不一定是样式表 stylesheets。
:::
#### `clientModules`
一组可在站点上全局加载的[客户端模块（Client modules）](https://www.docusaurus.cn/docs/advanced/client#client-modules)。
```javascript
module.exports = {
  clientModules: [
    require.resolve('./mySiteGlobalJs.js'),
    require.resolve('./mySiteGlobalCss.css'),
  ],
};
```
#### `ssrTemplate`

- 类型：`string`

以[ Eta 语法](https://eta.js.org/docs/syntax#syntax-overview)编写的 HTML 模板，将用于呈现您的应用程序。用于在 body 标签、meta 标签、自定义 viewport 等设置自定义属性。请注意，Docusaurus 将依赖正确构造的模板才能正常运行，一旦您自定义了它，您将必须确保您的模板符合上游的要求。
例子：
```javascript
module.exports = {
  ssrTemplate: `<!DOCTYPE html>
<html <%~ it.htmlAttributes %>>
  <head>
    <meta charset="UTF-8">
    <meta name="generator" content="Docusaurus v<%= it.version %>">
    <% it.metaAttributes.forEach((metaAttribute) => { %>
      <%~ metaAttribute %>
    <% }); %>
    <%~ it.headTags %>
    <% it.stylesheets.forEach((stylesheet) => { %>
      <link rel="stylesheet" href="<%= it.baseUrl %><%= stylesheet %>" />
    <% }); %>
    <% it.scripts.forEach((script) => { %>
      <link rel="preload" href="<%= it.baseUrl %><%= script %>" as="script">
    <% }); %>
  </head>
  <body <%~ it.bodyAttributes %>>
    <%~ it.preBodyTags %>
    <div id="__docusaurus">
      <%~ it.appHtml %>
    </div>
    <% it.scripts.forEach((script) => { %>
      <script src="<%= it.baseUrl %><%= script %>"></script>
    <% }); %>
    <%~ it.postBodyTags %>
  </body>
</html>`,
};
```
#### `titleDelimiter`

- 类型：`string`

用作生成的标题标签中的标题分隔符。
例如：
```javascript
module.exports = {
  titleDelimiter: '🦖', // Defaults to `|`
};
```
#### `baseUrlIssueBanner`

- 类型：`boolean`

启用后，将显示一个横幅，以防您的网站无法加载其 CSS 或 JavaScript 文件，这是一个非常常见的问题，通常与网站配置中的错误 baseUrl 有关。
例如：
```javascript
module.exports = {
  baseUrlIssueBanner: true, // Defaults to `true`
};
```
> 此横幅需要内联 CSS / JS，以防所有资源加载由于错误的基本 URL 而失败。如果您有严格的[内容安全策略](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)，则应该禁用它。

