---
layout: post
title:  "Vite 配置项"
date:   2025-11-21 14:14:00 +0800
---

* 目录
  {:toc #markdown-toc}

### 共享选项

**root**

项目根目录（index.html 文件所在的位置）。 默认： process.cwd()。

**base**

开发或生产环境服务的公共基础路径。默认： /。

**mode**

在配置中指明将会把 serve 和 build 时的模式 都 覆盖掉。默认： 'development' 用于开发，'production' 用于构建。

**define**

定义全局常量替换方式。其中每项在开发环境下会被定义在全局，而在构建时被静态替换。

```
export default defineConfig({
  define: {
    __APP_VERSION__: JSON.stringify('v1.0.0'),
    __API_URL__: 'window.__backend_api_url',
  },
})
```

**plugins**

需要用到的插件数组。

**publicDir**

作为静态资源服务的文件夹。默认： "public"。

**cacheDir**

存储缓存文件的目录。此目录下会存储预打包的依赖项或 vite 生成的某些缓存文件，使用缓存可以提高性能。默认： "node_modules/.vite"。

**resolve.alias**

将会被传递到 @rollup/plugin-alias 作为 entries 的选项。

它的核心作用是：为模块导入路径设置“快捷别名”，避免写冗长或脆弱的相对路径（如 ../../../../utils），提升代码可读性和可维护性。

它本质上是对 模块解析过程 的定制，底层由 @rollup/plugin-alias 实现。

```
resolve: {
  alias: [
    {
      find: /\$\//,
      replacement: `${pathResolve('src')}/`,
    },
  ]
} 
```

将所有以 $ 开头的导入路径（如 $/utils/helper）重写为 src/ 下的对应路径。

```
import { api } from '$/api/index'
// 实际解析为：
// import { api } from '/your-project/src/api/index'
```

**resolve.dedupe**

如果你在你的应用程序中有相同依赖的副本（比如 monorepos），请使用此选项强制 Vite 始终将列出的依赖项解析为同一副本（从项目根目录）。

假设你的项目结构如下：

```
my-monorepo/
├── node_modules/
│   └── vue@3.4.0          ← 根依赖
├── packages/
│   ├── web/                ← Vite 应用
│   │   └── node_modules/
│   │       └── vue@3.2.0   ← 子包局部安装（可能由其他依赖引入）
│   └── shared/
│       └── node_modules/
│           └── vue@3.4.0
```

当你在 web 包中运行 Vite 时：

- Vite 可能从 packages/web/node_modules/vue 加载 Vue 3.2.0

- 而 shared 模块使用的是 Vue 3.4.0

其结果是：同一个应用中存在两个 Vue 实例。项目运行时就会报错：

```
"Another instance of Vue is already running"
```

在 vite.config.js 中配置：

```
resolve: {
  dedupe: ['vue', 'pinia', 'axios'] // ← 关键！
}
```

Vite 在解析 vue 时，跳过局部 node_modules，直接从 项目根目录（my-monorepo/node_modules/vue）加载，从而确保整个应用使用 同一个 Vue 实例。

**resolve.conditions**

解决程序包中条件导出时的其他允许条件。

从 Node.js 12+ 开始，package.json 支持 exports 字段，允许包作者根据不同的使用环境提供不同的入口文件：

```
// node_modules/my-lib/package.json
{
  "name": "my-lib",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",     // ES 模块（给 Vite/现代打包工具）
      "require": "./dist/index.cjs",    // CommonJS（给 Node.js require）
      "browser": "./dist/browser.js",   // 浏览器专用版本
      "development": "./dist/dev.js",   // 开发环境
      "production": "./dist/prod.js"    // 生产环境
    }
  }
}
```

当你的代码 import 'my-lib' 时，打包工具会根据当前环境条件选择正确的入口。

Vite 在开发和构建时，会自动带上一组默认条件：

```
['module', 'browser', 'development']

// or
// ['module', 'browser', 'production']
```

这些条件告诉包管理器：“我是一个浏览器环境下的 ES 模块使用者，当前是开发/生产阶段”。

当某个库提供了自定义条件，但 Vite 默认不识别时，需要配置该项。

例如，一个库支持 "worker" 条件用于 Web Worker 环境：

```
{
  "exports": {
    ".": {
      "worker": "./dist/worker.js",
      "browser": "./dist/browser.js"
    }
  }
}
```

如果你在 Web Worker 中使用它，但 Vite 不知道 "worker" 条件，就会 fallback 到 "browser"，可能出错。

如下配置可解决此问题：

```
// vite.config.js
export default {
  resolve: {
    conditions: ['worker', 'module', 'browser'] // 把 'worker' 放前面提高优先级
  }
}
```

### resolve.mainFields

用于控制如何从 npm 包的 package.json 中选择入口文件的关键配置项。它决定了当你写 import 'some-lib' 时，Vite 到底加载哪个 JS 文件。

一个典型的 package.json 可能包含多个入口字段，针对不同环境：

```
{
  "name": "my-lib",
  "main": "dist/my-lib.cjs.js",        // ← CommonJS 入口（Node.js）
  "module": "dist/my-lib.esm.js",      // ← ES 模块入口（现代打包工具）
  "browser": "dist/my-lib.browser.js", // ← 浏览器专用版本（可能 polyfill Node API）
  "jsnext:main": "dist/index.mjs",     // ← 旧式 ESM 字段（已废弃，但仍有库用）
  "exports": { ... }                   // ← 条件导出（优先级高于 mainFields）
}
```

Vite 在浏览器端项目中的默认值为：

```
['browser', 'module', 'jsnext:main', 'jsnext']
```

解析顺序（优先级从高到低）：

- browser：优先使用浏览器优化版本（移除 Node.js 专属代码如 fs、path）

- module：使用 ES 模块版本（支持 tree-shaking）

- jsnext:main / jsnext：兼容旧生态（如早期 Rollup 生态）

如果某个包的 browser 字段指向“降级版”代码，你想用原生 ESM，则可以配置：

```
// vite.config.js
export default {
  resolve: {
    mainFields: ['module', 'browser'] // 优先用 module
  }
}
```

**resolve.extensions**

当你在代码中导入一个不带扩展名的模块时，Vite 会按照 resolve.extensions 中定义的顺序，依次尝试添加这些扩展名，直到找到匹配的文件为止。

默认： ['.mjs', '.js', '.mts', '.ts', '.jsx', '.tsx', '.json']

```
import { helper } from './utils'      // 没写 .js
```

Vite 会根据 resolve.extensions 依次查找：

```
./utils.mjs → 不存在
./utils.js → 找到！停止搜索
./api.mjs → 不存在
./api.js → 不存在
./api.mts → 不存在
./api.ts → 找到！
```

这样，就不用手动写扩展名，提升开发体验。

**resolve.preserveSymlinks**

启用此选项会使 Vite 通过原始文件路径（即不跟随符号链接的路径）而不是真正的文件路径（即跟随符号链接后的路径）确定文件身份。默认： false。

一般项目无需开启。

**html.cspNonce**

一个在生成脚本或样式标签时会用到的 nonce 值占位符。设置此值还会生成一个带有 nonce 值的 meta 标签。

在开发服务器（vite dev）和构建产物（vite build）中，自动为内联 `<script>` 和 `<style>` 标签注入 nonce 属性，并生成对应的 `<meta http-equiv="Content-Security-Policy">` 标签，以满足 CSP 的安全要求。

内容安全策略（CSP）是一种浏览器安全机制，用于防止 XSS（跨站脚本攻击）。它通过 HTTP 响应头或 `<meta>` 标签定义哪些资源可以加载。

例如，一个严格的 CSP 策略可能禁止所有内联脚本：

```
Content-Security-Policy: script-src 'self'
```

这会导致以下代码被浏览器阻止：

```
<script>console.log('inline')</script> <!-- 被阻止 -->
```

CSP 允许通过 nonce 机制放行特定内联脚本：

```
Content-Security-Policy: script-src 'self' 'nonce-abc123'
```

HTML 中对应脚本需带相同 nonce：

```
<script nonce="abc123">console.log('allowed')</script> <!-- 允许 -->
```

nonce 必须是每次请求随机生成的 base64 字符串，不可预测。

当在 vite.config.js 中设置：

```
export default {
  html: {
    cspNonce: 'your-nonce-value' // 通常由后端动态生成并传入
  }
}
```

Vite 会在处理 index.html 时：

- 为所有内联 <script> 和 <style> 注入 nonce 属性

- 自动生成 CSP meta 标签

其适用场景为：需要启用严格 CSP 策略的项目（尤其是传统后端 + Vite 前端混合架构）

**css.modules**

配置 CSS modules 的行为。选项将被传递给 postcss-modules。

CSS Modules 是一种 局部作用域 CSS 的方案，它通过将类名编译为全局唯一哈希值，避免样式污染。

```
/* button.module.css */
.primary {
  color: blue;
}
```

```
// Button.vue 或 .jsx
import styles from './button.module.css'
console.log(styles.primary) // → '_primary_xxx_123'（唯一哈希）
```

其结果是：该组件的 .primary 不会与其他组件的 .primary 冲突。

只要文件名以 .module.css / .module.scss 等结尾，Vite 就自动启用 CSS Modules。

常见配置项有：

```
export default {
  css: {
    modules: {
      // 1. 自定义生成的类名格式
      generateScopedName: '[name]__[local]___[hash:base64:5]',

      // 2. 全局类名（不被 hash）的白名单（支持正则）
      globalModulePaths: [/node_modules/],

      // 3. 是否压缩类名（生产环境默认开启）
      // （实际由 postcss-modules 控制）

      // 4. 自定义 getJSON 回调（用于提取样式映射）
      getJSON(cssFileName, json, outputFileName) {
        // 可用于生成 TypeScript 类型声明等
      },

      // 5. 启用作用域（默认 true）
      scopeBehaviour: 'local', // or 'global'

      // 6. 自定义 hash 函数（高级）
      hashPrefix: 'my-app',
    }
  }
}
```

通过该配置，可以让开发环境类名可读，以方便调试：

```
css: {
  modules: {
    generateScopedName: process.env.NODE_ENV === 'development'
      ? '[name]__[local]'
      : '[hash:base64:8]'
  }
}
```

- 开发时：Button__primary

- 生产时：aB3xK9mQ

**css.postcss**

内联的 PostCSS 配置（格式同 postcss.config.js），或者一个（默认基于项目根目录的）自定义的 PostCSS 配置路径。

该配置允许你：

- 内联定义 PostCSS 配置（无需单独的 postcss.config.js）

- 指定自定义的 PostCSS 配置文件路径

- 覆盖 Vite 内置的 PostCSS 行为（Vite 默认已集成 PostCSS，并会自动加载项目根目录的 postcss.config.js）

如果在 vite.config.js 中显式配置了 css.postcss，Vite 会优先使用你在 vite.config.js 中提供的 css.postcss 配置。 此时，项目根目录下的 postcss.config.js（或其它 PostCSS 配置文件）会被忽略。

如果没有在 vite.config.js 中配置 css.postcss，Vite 会自动查找并加载项目根目录下的 PostCSS 配置文件（如 postcss.config.js、.postcssrc.js、.postcssrc.json 等，遵循 PostCSS 官方的加载规则），这些配置将正常生效。

**css.preprocessorOptions**

指定传递给 CSS 预处理器的选项。文件扩展名用作选项的键。每个预处理器支持的选项可以在它们各自的文档中找到：

- sass/scss

- less

- styl/stylus

**css.preprocessorMaxWorkers**

如果启用了这个选项，那么 CSS 预处理器会尽可能在 worker 线程中运行。true 表示 CPU 数量减 1。默认： 0（不会创建任何 worker 线程，而是在主线程中运行）。

**css.devSourcemap**

在开发过程中是否启用 sourcemap。默认： false。

当设置为 true 时，Vite 在开发服务器（vite dev）中会为 CSS 文件（包括预处理语言如 SCSS/Less 编译后的 CSS）生成 Source Map。

这使得在浏览器开发者工具中调试样式时，能直接定位到原始源文件（如 .scss 或 .vue 中的 `<style>`），而不是编译后的 CSS。

**css.transformer**

该选项用于选择用于 CSS 处理的引擎。类型： 'postcss' | 'lightningcss'。默认： 'postcss'。

**css.lightningcss**

该选项用于配置 Lightning CSS。

**json.namedExports**

是否支持从 .json 文件中进行按名导入。默认值：true。

当启用时（默认），Vite 会把 JSON 文件的内容解析，并自动将顶层键作为具名导出，同时保留默认导出（整个 JSON 对象）。

```
// config.json
{
  "version": "1.0.0",
  "author": "Alice"
}
```

可以这样导出：

```
// 具名导入（named import）
import { version, author } from './config.json';

// 默认导入（default import）
import config from './config.json';

console.log(version); // "1.0.0"
console.log(config.author); // "Alice"
```

**json.stringify**

若设置为 true，导入的 JSON 会被转换为 export default JSON.parse("...")，这样会比转译成对象字面量性能更好，尤其是当 JSON 文件较大的时候。

类型： boolean | 'auto'。默认： 'auto'。

如果设置为 'auto'，只有当 数据大于 10kB 时，才会对数据进行字符串化处理。

```
{
  "users": [
    { "id": 1, "name": "Alice" },
    { "id": 2, "name": "Bob" }
    // ... 成千上万条数据
  ]
}
```

json.stringify: false 的情况下，Vite 将其转译为：

```
// 转译结果（对象字面量）
export default {
  users: [
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" }
    // ...
  ]
};
```

json.stringify: true（或大文件 + 'auto'）的情况下，Vite 将其转译为：

```
// 转译结果（字符串 + JSON.parse）
export default JSON.parse('{\n  "users": [\n    { "id": 1, "name": "Alice" },\n    { "id": 2, "name": "Bob" }\n  ]\n}');
```

显然，后者在大型 JSON 场景下启动更快、内存占用更低。

**esbuild**

esbuild 配置项（类型：ESBuildOptions | false）主要用于自定义 esbuild 对源码的转译行为。Vite 默认使用 esbuild 来快速转译 JavaScript/TypeScript（包括 JSX），而这个配置允许你覆盖其默认选项。

**assetsInclude**

在 Vite 中，默认只将某些扩展名（如 .png, .jpg, .svg, .woff2 等）视为静态资源（asset）。当你通过 JavaScript 导入这些文件时，Vite 会返回其解析后的公共 URL（例如 /assets/logo.abc123.png）；当在 HTML 中直接引用或通过 fetch() 请求时，Vite 开发服务器也会正确提供它们。

如果你有非标准扩展名的资源文件（比如 .gltf, .hdr, .bin, .txt, .csv 等），Vite 默认不会把它们当作静态资源处理，可能会：

- 尝试用插件去解析（导致报错）

- 在导入时无法返回正确 URL

- 开发服务器无法正确响应请求

这时候就需要用 assetsInclude 显式告诉 Vite：这些也是静态资源，请跳过转换管道，直接当作 asset 处理。

```
// vite.config.js
export default {
  assetsInclude: [
    '**/*.gltf',
    '**/*.hdr',
    /\.bin$/,
    'data/*.csv'
  ]
}
```

**logLevel**

用于控制开发服务器（dev server）和构建过程（build）中输出到控制台的日志级别。

类型： 'info' | 'warn' | 'error' | 'silent'，默认为 'info'。

**customLogger**

使用自定义 logger 记录消息。可以使用 Vite 的 createLogger API 获取默认的 logger 并对其进行自定义，例如，更改消息或过滤掉某些警告。

**clearScreen**

设为 false 可以避免 Vite 清屏而错过在终端中打印某些关键信息。默认： true

**envDir**

用于加载 .env 文件的目录。可以是一个绝对路径，也可以是相对于项目根的路径。设置为 false 将禁用 .env 文件的加载。默认： root。

**envPrefix**

在 Vite 中，只有以特定前缀开头的环境变量才会被嵌入到客户端代码中，并通过 import.meta.env 暴露给前端使用。默认值：'VITE_'

```
export default {
  envPrefix: 'VITE_'
}
```

也可以接受多个前缀：

export default {
envPrefix: ['VITE_', 'PUBLIC_'] // 两个前缀都允许
}

这是 Vite 的一项重要安全设计：

- 如果没有前缀限制，开发者可能不小心把 .env 中的密钥（如 AWS_SECRET_KEY）通过 import.meta.env 泄露到浏览器中。

- 强制使用前缀（如 VITE_）是一种显式声明“此变量可公开” 的约定。

**appType**

应用类型。类型： 'spa' | 'mpa' | 'custom'。默认： 'spa'。

- 'spa'：包含 HTML 中间件以及使用 SPA 回退。在预览中将 sirv 配置为 single: true

- 'mpa'：包含 HTML 中间件

- 'custom'：不包含 HTML 中间件

**future**

启用未来的重大变更，为顺利迁移到 Vite 的下一个主要版本做好准备。随着新功能的开发，这个列表可能会随时进行更新、添加或移除。

### 服务器选项

**server.host**

控制开发服务器绑定到哪个主机地址。

| 值                     | 行为                                                   |
| --------------------- | ---------------------------------------------------- |
| `'localhost'`（默认）     | 仅监听本地回环地址（127.0.0.1），只能本机访问。                         |
| `'127.0.0.1'`         | 同上，明确指定 IPv4 回环地址。                                   |
| `'::1'`               | IPv6 的回环地址（等效于 localhost 的 IPv6 版本）。                 |
| `'0.0.0.0'`           | 监听所有 IPv4 网络接口，允许局域网内其他设备通过本机 IP 访问（如`192.168.x.x`）。 |
| `true`                | 等效于`'0.0.0.0'`。                    |
| `'192.168.1.100'`（示例） | 仅监听指定的本地 IP 地址（需该 IP 存在于本机网卡上）。                      |

**server.allowedHosts**

Vite 开发服务器中的一个安全配置项，用于控制哪些 Host 请求头（HTTP Host header） 被视为合法，从而防止 Host 头攻击。

当你通过以下方式访问开发服务器时：

- http://192.168.1.100:5173

- http://myapp.test

- 通过反向代理（如 nginx）转发请求

浏览器或代理会发送 Host: xxx 请求头。Vite 会检查该 Host 是否在允许列表中。若不在且未匹配默认规则，会返回 403 Forbidden 或空白页面。

**server.port**

指定开发服务器端口。注意：如果端口已经被使用，Vite 会自动尝试下一个可用的端口，所以这可能不是开发服务器最终监听的实际端口。默认值： 5173。

**server.strictPort**

设为 true 时若端口已被占用则会直接退出，而不是尝试下一个可用端口。

**server.https**

Vite 开发服务器中用于启用 HTTPS（TLS）和 HTTP/2 的配置项。

类型：boolean | https.ServerOptions。默认值：false（即使用 HTTP）。

```
import fs from 'fs'
import path from 'path'

export default {
  server: {
    https: {
      key: fs.readFileSync(path.resolve(__dirname, 'certs/private.key')),
      cert: fs.readFileSync(path.resolve(__dirname, 'certs/certificate.crt'))
    }
  }
}
```

**server.open**

开发服务器启动时，自动在浏览器中打开应用程序。类型： boolean | string。当该值为字符串时，它将被用作 URL 的路径名。

```
export default defineConfig({
  server: {
    open: '/docs/index.html',
  },
})
```

**server.proxy**

Vite 开发服务器中用于配置 HTTP 代理 的核心选项，主要用于解决开发时的 跨域（CORS）问题，或将 API 请求转发到后端服务。

在开发阶段，前端运行在如 http://localhost:5173，而后端 API 可能在 http://localhost:3000 或远程服务器上。浏览器出于安全限制会阻止跨域请求。

Vite 的 proxy 功能让你在开发服务器上“代理”这些请求，使它们看起来像是同源的。

```
type ProxyOptions = {
  target: string
  changeOrigin?: boolean
  secure?: boolean
  ws?: boolean
  rewrite?: (path: string) => string
  configure?: (proxy: any, options: ProxyOptions) => void
  // ...其他 http-proxy-middleware 支持的选项
}

server: {
  proxy: Record<string, string | ProxyOptions>
}
```

**server.cors**

Vite 开发服务器中用于配置 跨域资源共享（CORS）策略 的选项。它控制哪些**外部源（origin）**可以访问 Vite 开发服务器上的资源（如 HTML、JS、CSS、HMR 连接等）。

默认： { origin: /^https?:\/\/(?:(?:[^:]+\.)?localhost|127\.0\.0\.1|\[::1\])(?::\d+)?$/ } （允许 localhost、127.0.0.1 和 ::1）

设置为true，将允许所有源（开发调试可用，但有风险）。

**server.headers**

指定服务器响应的 header。

**server.hmr**

Vite 开发服务器中用于配置 热模块替换（Hot Module Replacement, HMR） 连接行为的选项。

HMR 是 Vite 实现“保存即更新”、无需刷新页面的核心机制，它通过 WebSocket 与浏览器客户端保持通信。

类型：boolean | HmrOptions。默认值：true（启用 HMR，自动使用当前开发服务器的协议/主机/端口）

**server.warmup**

在开发服务器启动时预先转换（transform）并缓存指定的模块，从而加快首次页面加载速度，避免“转换瀑布”（transform waterfall） —— 即浏览器请求 A → A 依赖 B → B 依赖 C …… 串行等待编译的问题。

假设你的应用入口是 src/main.ts，它依赖大量组件和库。首次访问时，Vite 需要逐个编译这些文件，造成白屏时间长。

```
// vite.config.js
export default {
  server: {
    warmup: {
      clientFiles: [
        'src/main.ts',
        'src/App.vue',
        'src/router/index.ts',
        'src/stores/**/*',      // 支持 glob 模式（需使用 fast-glob）
        'src/views/HomeView.vue'
      ]
    }
  }
}
```

Vite 会递归解析这些文件的依赖，并提前编译整个子图（subgraph）。

**server.watch**

Vite 服务器的文件监听器默认会监听 root 目录，同时会跳过 .git/、node_modules/，以及 Vite 的 cacheDir 和 build.outDir 这些目录。

当监听到文件更新时，Vite 会应用 HMR 并且只在需要时更新页面。如果设置为 null，则不会监视任何文件。

**server.middlewareMode**

以中间件模式创建 Vite 服务器，亦即，将 Vite 开发服务器作为 自定义 HTTP 服务器（如 Express、Koa、Fastify 等）的中间件来使用，而不是启动一个独立的服务器。

**server.fs.strict**

用于增强文件系统安全的重要配置项，旨在防止开发服务器意外暴露项目根目录之外的敏感文件。

**server.fs.allow**

限制哪些文件可以通过 /@fs/ 路径提供服务。当 server.fs.strict 设置为 true 时，访问这个目录列表外的文件将会返回 403 结果。

**server.fs.deny**

用于限制 Vite 开发服务器提供敏感文件的黑名单。这会比 server.fs.allow 选项的优先级更高。

**server.origin**

用于定义开发调试阶段生成资源的 origin。

**server.sourcemapIgnoreList**

用于指定是否忽略服务器 sourcemap 中的源文件。

### 构建选项

**build.target**

用于指定输出代码的 JavaScript 语法目标（即浏览器兼容性） 的关键选项。它直接影响 esbuild（或 Babel，若使用）对代码的转译程度。

其作用是：告诉打包工具“目标运行环境支持哪些 JS 特性”，从而决定是否需要将新语法（如 ??、?.、class fields）转译为旧语法。

默认值是：'modules'，它指示：只支持能原生运行 ES 模块（ESM）的现代浏览器。Vite 将替换 modules 为具体浏览器目标：

```
['es2020', 'edge88', 'firefox78', 'chrome87', 'safari14']
```

这些浏览器均支持：`<script type="module">`，动态 import()，import.meta。

值 'esnext' 指示：假设运行环境支持最新 JS 语法。亦即，几乎不做语法转译。其潜在的风险是：生成的代码可能无法在当前任何浏览器中直接运行。

**build.modulePreload**

在生产构建（vite build）中用于控制模块预加载（module preload）行为的配置项，主要影响现代浏览器如何高效加载动态导入（import()）的代码分割块（chunks）。

在支持 ES Modules 的现代浏览器中：

- 当你使用动态导入 `const mod = await import('./lazy.js')` 时，浏览器需要先下载并解析 lazy.js。

- 为了提前告知浏览器“稍后会用到这些模块”，可以通过 `<link rel="modulepreload">` 提前加载它们，避免请求瀑布（waterfall）。

```
<link rel="modulepreload" href="/assets/LazyView-abc123.js">
```

这能显著提升懒加载路由、组件的首次交互速度。

build.modulePreload 的核心功能：

- 自动注入 `<link rel="modulepreload">`，Vite 在构建时会：分析所有动态导入（如 Vue 的 `() => import('./views/Home.vue')`）， 在 HTML 中为这些 chunk 自动生成`<link rel="modulepreload">`标签。

- 启用 Polyfill，并非所有浏览器都支持 `<link rel="modulepreload">`（例如旧版 Edge），Vite 会向每个入口 HTML 注入一个轻量级 polyfill 脚本，该 polyfill 会将 modulepreload 转换为普通的 `<script type="module">` + 缓存，模拟预加载行为。

- 自定义 resolveDependencies，覆盖 Vite 默认的依赖解析逻辑，例如：排除某些 chunk（如超大组件，不想预加载），添加额外依赖（如手动管理的动态资源）。

```
// vite.config.js
export default {
  build: {
    modulePreload: {
      resolveDependencies: (filename, deps, { hostId, hostType }) => {
        // 过滤掉大于 500KB 的 chunk
        return deps.filter(dep => dep.size < 500 * 1024)
      }
    }
  }
}
```

**build.outDir**

指定输出路径（相对于项目根目录)。

**build.assetsDir**

指定生成静态资源的存放路径（相对于 build.outDir）。

**build.assetsInlineLimit**

小于此阈值的导入或引用资源将内联为 base64 编码，以避免额外的 http 请求。设置为 0 可以完全禁用此项。默认： 4096 (4 KiB)。

**build.cssCodeSplit**

启用/禁用 CSS 代码拆分。当启用（默认）时，在异步 chunk 中导入的 CSS 将内联到异步 chunk 本身，并在其被加载时一并获取。

如果禁用，整个项目中的所有 CSS 将被提取到一个 CSS 文件中。

**build.cssTarget**

为什么除了 build.target 以外，还需要 build.cssTarget 呢？因为：JavaScript 和 CSS 的浏览器兼容性是分开演进的。

有些浏览器（尤其是嵌入式 WebView，如微信、QQ、旧版 Android 浏览器）：支持现代 JS 语法（如 async/await, 箭头函数），但不支持某些现代 CSS 特性（如 #RGBA 颜色、aspect-ratio、:focus-visible）。

该配置只影响 CSS 的压缩与降级行为（如颜色格式、语法转换、厂商前缀等）。

**build.cssMinify**

此选项允许用户覆盖 CSS 最小化压缩的配置，而不是使用默认的 build.minify，这样你就可以单独配置 JS 和 CSS 的最小化压缩方式。

类型： boolean | 'esbuild' | 'lightningcss'。（即不同的 css 压缩器）

对于客户端，其默认值与 build.minify 相同；对于 SSR，其默认值为 'esbuild'。

**build.sourcemap**

构建后是否生成 source map 文件。

**build.rollupOptions**

自定义底层的 Rollup 打包配置。

**build.commonjsOptions**

传递给 @rollup/plugin-commonjs 插件的选项。

**build.dynamicImportVarsOptions**

传递给 @rollup/plugin-dynamic-import-vars 的选项。

**build.lib**

以库的形式构建。

```
import { defineConfig } from 'vite'

export default defineConfig({
  build: {
    lib: {
      entry: ['src/main.js'],
      fileName: (format, entryName) => `my-lib-${entryName}.${format}.js`,
      cssFileName: 'my-lib-style',
    },
  },
})
```

**build.manifest**

指示是否生成一个 manifest 文件，包含了没有被 hash 过的资源文件名和 hash 后版本的映射，然后服务器框架可使用该映射来呈现正确的资源引入链接。

**build.ssrManifest**

指示是否生成 SSR 的 manifest 文件，以确定生产中的样式链接与资源预加载指令。

**build.ssr**

指示是否面向 SSR 进行构建。

**build.emitAssets**

用于 非客户端构建（如 SSR、库模式、中间层服务）中强制输出静态资源文件（如图片、字体、CSS、JS chunk 等）。默认 false。

在 SSR 构建（vite build --ssr）时，Vite 只生成 服务端入口 bundle（如 dist/server/entry-server.js），不会输出：图片（.png, .jpg），字体（.woff2），样式文件（.css），客户端 JS chunk（异步路由、组件等）。

**build.minify**

设置为 false 可以禁用最小化混淆，或是用来指定使用哪种混淆器。默认为 Esbuild，它比 terser 快 20-40 倍，压缩率只差 1%-2%。

类型： boolean | 'terser' | 'esbuild'。客户端构建默认为'esbuild'，SSR构建默认为 false。

**build.terserOptions**

传递给 Terser 的更多 minify 选项。

**build.write**

设置为 false 来禁用将构建后的文件写入磁盘。这常用于编程式地调用 build() 在写入磁盘之前，需要对构建后的文件进行进一步处理。

**build.emptyOutDir**

默认情况下，若 outDir 在 root 目录下，则 Vite 会在构建时清空该目录。若 outDir 在根目录之外则会抛出一个警告避免意外删除掉重要的文件。可以设置该选项来关闭这个警告。

**build.copyPublicDir**

默认情况下，Vite 会在构建阶段将 publicDir 目录中的所有文件复制到 outDir 目录中。可以通过设置该选项为 false 来禁用该行为。

**build.reportCompressedSize**

启用/禁用 gzip 压缩大小报告。压缩大型输出文件可能会很慢，因此禁用该功能可能会提高大型项目的构建性能。默认： true。

**build.chunkSizeWarningLimit**

规定触发警告的 chunk 大小。（以 kB 为单位）。默认： 500。

**build.watch**

在构建模式下启用文件监听（watch mode） 。它允许你在运行 vite build 时，自动重新构建当源文件发生变化，而无需手动重复执行构建命令。默认值：null（即不监听，构建一次就退出）。

### 预览选项

**preview.host**

为预览服务器指定 ip 地址。 设置为 0.0.0.0 或 true 会监听所有地址，包括局域网和公共地址。

**preview.allowedHosts**

Vite 允许响应的主机名。

**preview.port**

为预览服务器指定端口。注意，如果设置的端口已被使用，Vite 将自动尝试下一个可用端口，所以这可能不是最终监听的服务器端口。默认： 4173。

**preview.strictPort**

设置为 true 时，如果端口已被使用，则直接退出，而不会再进行后续端口的尝试。

**preview.https**

启用 TLS + HTTP/2。

**preview.open**

预览服务器启动时，自动在浏览器中打开应用程序。

**preview.proxy**

为预览服务器配置自定义代理规则。

**preview.cors**

为预览服务器配置 CORS。

**preview.headers**

指明服务器返回的响应头。

### 依赖优化选项

**optimizeDeps.entries**

默认情况下，Vite 会抓取你的 index.html 来检测需要预构建的依赖项（忽略了node_modules、build.outDir、__tests__ 和 coverage）。

如果指定了 build.rollupOptions.input，Vite 将转而去抓取这些入口点。

**optimizeDeps.exclude**

在预构建中强制排除的依赖项。

**optimizeDeps.include**

默认情况下，不在 node_modules 中的，链接的包不会被预构建。使用此选项可强制预构建链接的包。

**optimizeDeps.esbuildOptions**

在依赖扫描和优化过程中传递给 esbuild 的选项。

**optimizeDeps.force**

设置为 true 可以强制依赖预构建，而忽略之前已经缓存过的、已经优化过的依赖。

**optimizeDeps.holdUntilCrawlEnd**

当该功能被启用时，系统会在冷启动时保持第一个优化的依赖结果，直到所有的静态导入都被检索完毕。
