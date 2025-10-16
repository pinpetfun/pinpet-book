# Docsify 多语言文档实现指南

本文档详细介绍如何在 Docsify 中实现多语言支持，帮助你构建国际化的技术文档站点。

## 目录

- [实现原理](#实现原理)
- [文件夹结构](#文件夹结构)
- [配置方法](#配置方法)
- [语言切换器](#语言切换器)
- [完整示例](#完整示例)
- [参考案例](#参考案例)

---

## 实现原理

Docsify 通过 **URL Hash 路由** 实现多语言支持：

- 默认语言：`https://your-site.com/#/`
- 中文：`https://your-site.com/#/zh-cn/`
- 英文：`https://your-site.com/#/en/`
- 西班牙语：`https://your-site.com/#/es/`

每个语言对应一个独立的文件夹，包含该语言的所有文档内容。

---

## 文件夹结构

### 方案一：单仓库多语言（推荐）

```
docs/
├── index.html           # Docsify 配置文件
├── README.md            # 默认语言首页（如英文）
├── _sidebar.md          # 默认语言侧边栏
├── guide.md             # 默认语言其他页面
├── zh-cn/              # 中文文档
│   ├── README.md       # 中文首页
│   ├── _sidebar.md     # 中文侧边栏
│   └── guide.md        # 中文指南
├── es/                 # 西班牙语文档
│   ├── README.md
│   ├── _sidebar.md
│   └── guide.md
└── ja/                 # 日语文档
    ├── README.md
    ├── _sidebar.md
    └── guide.md
```

### 方案二：多仓库分离

将不同语言的文档放在不同的 Git 仓库，通过 `alias` 配置引用：

```
docs/          # 英文文档（主仓库）
docs-zh/       # 中文文档（独立仓库）
docs-es/       # 西班牙语文档（独立仓库）
```

---

## 配置方法

### 1. 基础配置

在 `index.html` 中添加多语言配置：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>My Documentation</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify@4/lib/themes/vue.css">
</head>
<body>
  <div id="app"></div>
  <script>
    window.$docsify = {
      name: 'My Project',
      repo: 'https://github.com/your-repo',
      loadSidebar: true,
      auto2top: true,

      // 多语言搜索支持
      search: {
        paths: 'auto',
        placeholder: {
          '/zh-cn/': '搜索',
          '/es/': 'Buscar',
          '/': 'Search'
        },
        noData: {
          '/zh-cn/': '找不到结果',
          '/es/': '¡No hay resultados!',
          '/': 'No results!'
        },
        pathNamespaces: ['/zh-cn', '/es', '/ja']
      }
    };
  </script>
  <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
  <script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/search.min.js"></script>
</body>
</html>
```

### 2. 多仓库别名配置

如果使用多仓库方案，添加 `alias` 配置：

```javascript
window.$docsify = {
  alias: {
    '/zh-cn/(.*)': 'https://raw.githubusercontent.com/your-repo/docs-zh/master/$1',
    '/es/(.*)': 'https://raw.githubusercontent.com/your-repo/docs-es/master/$1',
    '/ja/(.*)': 'https://raw.githubusercontent.com/your-repo/docs-ja/master/$1',
  }
};
```

### 3. 自动语言检测插件

自动为 HTML 标签设置正确的 `lang` 属性：

```javascript
window.$docsify = {
  plugins: [
    function(hook, vm) {
      hook.beforeEach(function(html) {
        // 从 URL 中提取语言代码
        var lang = location.hash.match(/#\/(zh-cn|es|ja)\//);
        if (lang) {
          document.documentElement.setAttribute('lang', lang[1]);
        } else {
          document.documentElement.setAttribute('lang', 'en');
        }
        return html;
      });
    }
  ]
};
```

---

## 语言切换器

### 方法一：Navbar 导航栏

在 `_navbar.md` 中添加语言切换链接：

```markdown
* 语言切换
  * [English](/)
  * [中文](/zh-cn/)
  * [Español](/es/)
  * [日本語](/ja/)
```

在 `index.html` 中启用导航栏：

```javascript
window.$docsify = {
  loadNavbar: true
};
```

### 方法二：自定义插件下拉菜单

添加一个更优雅的语言切换下拉菜单：

```javascript
window.$docsify = {
  plugins: [
    function(hook, vm) {
      hook.mounted(function() {
        // 创建语言切换器
        var languageSwitcher = `
          <div class="language-switcher">
            <select onchange="window.location.hash = this.value">
              <option value="#/" ${!location.hash.includes('/zh-cn/') && !location.hash.includes('/es/') ? 'selected' : ''}>🇬🇧 English</option>
              <option value="#/zh-cn/" ${location.hash.includes('/zh-cn/') ? 'selected' : ''}>🇨🇳 中文</option>
              <option value="#/es/" ${location.hash.includes('/es/') ? 'selected' : ''}>🇪🇸 Español</option>
              <option value="#/ja/" ${location.hash.includes('/ja/') ? 'selected' : ''}>🇯🇵 日本語</option>
            </select>
          </div>
        `;

        document.querySelector('.sidebar').insertAdjacentHTML('afterbegin', languageSwitcher);
      });
    }
  ]
};
```

添加样式：

```html
<style>
  .language-switcher {
    padding: 10px;
    text-align: center;
  }

  .language-switcher select {
    padding: 8px 12px;
    border: 1px solid #ddd;
    border-radius: 4px;
    background: white;
    cursor: pointer;
    font-size: 14px;
  }

  .language-switcher select:hover {
    border-color: #42b983;
  }
</style>
```

---

## 完整示例

### 项目结构

```
my-docs/
├── index.html
├── _navbar.md
├── README.md
├── _sidebar.md
├── guide/
│   └── quickstart.md
└── zh-cn/
    ├── README.md
    ├── _sidebar.md
    └── guide/
        └── quickstart.md
```

### index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>My Docs - Multi-Language</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify@4/lib/themes/vue.css">
  <style>
    .language-switcher {
      padding: 10px;
      text-align: center;
      border-bottom: 1px solid #eee;
    }
    .language-switcher select {
      padding: 8px 12px;
      border: 1px solid #ddd;
      border-radius: 4px;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <div id="app"></div>
  <script>
    window.$docsify = {
      name: 'My Documentation',
      repo: 'https://github.com/your-repo/your-project',
      loadSidebar: true,
      loadNavbar: true,
      auto2top: true,
      maxLevel: 4,
      subMaxLevel: 3,

      // 多语言搜索配置
      search: {
        paths: 'auto',
        placeholder: {
          '/zh-cn/': '搜索文档',
          '/es/': 'Buscar',
          '/': 'Search'
        },
        noData: {
          '/zh-cn/': '找不到结果！',
          '/es/': '¡No hay resultados!',
          '/': 'No results!'
        },
        pathNamespaces: ['/zh-cn', '/es']
      },

      // 自定义插件
      plugins: [
        // 语言检测
        function(hook, vm) {
          hook.beforeEach(function(html) {
            var lang = location.hash.match(/#\/(zh-cn|es)\//);
            document.documentElement.setAttribute('lang', lang ? lang[1] : 'en');
            return html;
          });
        },

        // 语言切换器
        function(hook, vm) {
          hook.mounted(function() {
            var languageSwitcher = `
              <div class="language-switcher">
                <select onchange="window.location.hash = this.value + window.location.hash.replace(/^#\/(zh-cn\/|es\/)?/, '')">
                  <option value="#/" ${!location.hash.includes('/zh-cn/') && !location.hash.includes('/es/') ? 'selected' : ''}>🇬🇧 English</option>
                  <option value="#/zh-cn/" ${location.hash.includes('/zh-cn/') ? 'selected' : ''}>🇨🇳 简体中文</option>
                  <option value="#/es/" ${location.hash.includes('/es/') ? 'selected' : ''}>🇪🇸 Español</option>
                </select>
              </div>
            `;
            document.querySelector('.sidebar').insertAdjacentHTML('afterbegin', languageSwitcher);
          });
        }
      ]
    };
  </script>

  <!-- Docsify 核心库 -->
  <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>

  <!-- 搜索插件 -->
  <script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/search.min.js"></script>

  <!-- 其他插件 -->
  <script src="//cdn.jsdelivr.net/npm/docsify-copy-code@2"></script>
  <script src="//cdn.jsdelivr.net/npm/prismjs@1/components/prism-bash.min.js"></script>
  <script src="//cdn.jsdelivr.net/npm/prismjs@1/components/prism-javascript.min.js"></script>
</body>
</html>
```

### README.md (英文首页)

```markdown
# Welcome to My Documentation

> A comprehensive guide for developers

This documentation is available in multiple languages:
- [English](/) (current)
- [简体中文](/zh-cn/)
- [Español](/es/)

## Quick Start

Get started with our [Quick Start Guide](/guide/quickstart).
```

### zh-cn/README.md (中文首页)

```markdown
# 欢迎来到我的文档

> 为开发者提供的全面指南

本文档提供多语言版本：
- [English](/)
- [简体中文](/zh-cn/) (当前)
- [Español](/es/)

## 快速开始

查看我们的[快速开始指南](/zh-cn/guide/quickstart)。
```

### _sidebar.md (英文侧边栏)

```markdown
* [Home](/)
* [Guide](/guide/quickstart)
* [API Reference](/api/)
```

### zh-cn/_sidebar.md (中文侧边栏)

```markdown
* [首页](/zh-cn/)
* [指南](/zh-cn/guide/quickstart)
* [API 参考](/zh-cn/api/)
```

---

## 参考案例

### 1. Markdown Preview Enhanced

示例网站：https://shd101wyy.github.io/markdown-preview-enhanced/#/

**特点：**
- 使用文件夹结构管理多语言（zh-cn, zh-tw, ja-jp 等）
- 侧边栏顶部提供语言切换下拉菜单
- 每个语言拥有完全独立的侧边栏配置

### 2. Docsify 官方文档

**特点：**
- 使用多仓库方案
- 通过 `alias` 从不同 GitHub 仓库加载内容
- 提供 10+ 种语言版本

### 3. Vue.js 文档

**特点：**
- 每个语言独立域名（如 cn.vuejs.org）
- 使用 VitePress（Docsify 的替代方案）
- 完全本地化的内容和 UI

---

## 最佳实践

### 1. 内容同步

- **推荐**：使用主语言（如英文）作为源文档
- 通过 Git 分支或 tags 管理翻译版本
- 使用工具（如 Crowdin）管理翻译流程

### 2. URL 设计

- **保持一致**：所有语言使用相同的文件路径结构
- **示例**：
  - English: `#/guide/quickstart`
  - Chinese: `#/zh-cn/guide/quickstart`
  - Spanish: `#/es/guide/quickstart`

### 3. 默认语言选择

方法一：根据浏览器语言自动跳转

```javascript
window.$docsify = {
  plugins: [
    function(hook, vm) {
      hook.init(function() {
        // 仅在首次访问时检测
        if (location.hash === '#/' && !sessionStorage.getItem('lang-detected')) {
          var userLang = navigator.language || navigator.userLanguage;
          if (userLang.startsWith('zh')) {
            location.hash = '#/zh-cn/';
          } else if (userLang.startsWith('es')) {
            location.hash = '#/es/';
          }
          sessionStorage.setItem('lang-detected', 'true');
        }
      });
    }
  ]
};
```

方法二：记住用户选择

```javascript
// 保存用户选择的语言
function saveLangPreference(lang) {
  localStorage.setItem('preferred-lang', lang);
}

// 恢复用户选择
window.$docsify = {
  plugins: [
    function(hook, vm) {
      hook.init(function() {
        var preferredLang = localStorage.getItem('preferred-lang');
        if (preferredLang && location.hash === '#/') {
          location.hash = preferredLang === 'en' ? '#/' : `#/${preferredLang}/`;
        }
      });
    }
  ]
};
```

### 4. SEO 优化

在每个语言的 `index.html` 中添加正确的 `lang` 属性和 meta 标签：

```html
<html lang="zh-CN">
<head>
  <meta name="description" content="你的文档描述">
  <meta property="og:locale" content="zh_CN">
  <link rel="alternate" hreflang="en" href="https://your-site.com/#/">
  <link rel="alternate" hreflang="zh-CN" href="https://your-site.com/#/zh-cn/">
</head>
```

---

## 常见问题

### Q1: 如何处理图片等静态资源？

**方案 1**：共享静态资源

```
docs/
├── images/         # 所有语言共享
│   └── logo.png
├── README.md       # English: ![Logo](images/logo.png)
└── zh-cn/
    └── README.md   # Chinese: ![Logo](../images/logo.png)
```

**方案 2**：独立静态资源

```
docs/
├── images/
│   └── logo.png
└── zh-cn/
    ├── images/
    │   └── logo.png    # 中文特定图片
    └── README.md
```

### Q2: 搜索功能无法搜索所有语言？

确保配置了 `pathNamespaces`：

```javascript
search: {
  pathNamespaces: ['/zh-cn', '/es'],  // 添加所有语言路径
}
```

### Q3: 语言切换后回到首页？

改进语言切换器，保持当前页面路径：

```javascript
function switchLanguage(newLang) {
  var currentPath = location.hash.replace(/^#\/(zh-cn\/|es\/)?/, '');
  var newHash = newLang === 'en' ? `#/${currentPath}` : `#/${newLang}/${currentPath}`;
  location.hash = newHash;
}
```

---

## 总结

Docsify 多语言支持灵活且强大，适合从小型项目到大型文档站点。选择合适的方案：

- **小型项目**：单仓库 + 文件夹结构
- **大型项目**：多仓库 + alias 配置
- **企业级**：结合 CI/CD 自动化翻译和部署

参考本指南，你可以快速为 PinPet SDK 文档项目添加多语言支持！
