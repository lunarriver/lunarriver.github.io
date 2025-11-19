---
layout: post
title:  "Cross Origin Q&A"
date:   2025-11-17 15:16:00 +0800
---

* 目录
{:toc #markdown-toc}

### 参考链接

- https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Guides/CORS#%E8%8B%A5%E5%B9%B2%E8%AE%BF%E9%97%AE%E6%8E%A7%E5%88%B6%E5%9C%BA%E6%99%AF

- https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy

- https://fetch.spec.whatwg.org/#origin-header

- https://developer.mozilla.org/zh-CN/docs/Web/HTML/Reference/Attributes/crossorigin

### 同源策略

同源策略是一个重要的安全策略，它用于限制一个源的文档或者它加载的脚本如何能与另一个源的资源进行交互。

如果两个 URL 的**协议**、**端口**（如果有指定的话）和**主机**都相同的话，则这两个 URL 是同源的。这个方案也被称为“协议/主机/端口元组”，或者直接是“元组”。

下表给出了与 URL http://store.company.com/dir/page.html 的源进行对比的示例：

| URL                                               | 结果  | 原因                      |
| ------------------------------------------------- | --- | ----------------------- |
| `http://store.company.com/dir2/other.html`        | 同源  | 只有路径不同                  |
| `http://store.company.com/dir/inner/another.html` | 同源  | 只有路径不同                  |
| `https://store.company.com/secure.html`           | 失败  | 协议不同                    |
| `http://store.company.com:81/dir/etc.html`        | 失败  | 端口不同（`http://`默认端口是 80） |
| `http://news.company.com/dir/other.html`          | 失败  | 主机不同                    |

通过 document.domain setter **修改源**的方式已被弃用，因为它破坏了同源策略所提供的安全保护，并使浏览器中的源模型复杂化，导致互操作性问题和安全漏洞。

### 跨源网络访问

同源策略控制不同源之间的交互，例如在使用 XMLHttpRequest 或 <img> 标签时则会受到同源策略的约束。这些交互通常分为三类：

- 跨源写操作（Cross-origin writes）一般是被允许的。例如链接、重定向以及表单提交。特定少数的 HTTP 请求需要添加预检请求。 

- 跨源资源嵌入（Cross-origin embedding）一般是被允许的（后面会举例说明）。

- 跨源读操作（Cross-origin reads）一般是不被允许的，但常可以通过内嵌资源来巧妙的进行读取访问。例如，你可以读取嵌入图片的高度和宽度，调用内嵌脚本的方法，或得知内嵌资源的可用性。

以下是可能嵌入跨源的资源的一些示例：

- 使用 `<script src="…"></script>` 标签嵌入的 JavaScript 脚本。语法错误信息只能被同源脚本中捕捉到。

- 使用 `<link rel="stylesheet" href="…">` 标签嵌入的 CSS。由于 CSS 的松散的语法规则，CSS 的跨源需要一个设置正确的 Content-Type 标头。如果样式表是跨源的，且 MIME 类型不正确，资源不以有效的 CSS 结构开始，浏览器会阻止它的加载。

- 通过 `<img>` 展示的图片。

- 通过 `<video>` 和 `<audio>` 播放的多媒体资源。

- 通过 `<object>` 和 `<embed>` 嵌入的插件。

- 通过 @font-face 引入的字体。一些浏览器允许跨源字体（cross-origin fonts），另一些需要同源字体（same-origin fonts）。

- 通过 `<iframe>` 载入的任何资源。站点可以使用 X-Frame-Options 标头来阻止这种形式的跨源交互。

### 为什么跨源写操作反而比跨域读操作具有更大的自由度？

答案在于：Web 安全机制的设计目标不是防止“请求被发送”，而是防止“敏感数据被窃取”。

- 写操作本身不泄露信息给攻击者，而因为读操作会把目标网站的响应内容暴露给攻击者的 JavaScript。

- 服务端系统必须防御 CSRF（跨站请求伪造） —— 通过 CSRF Token、SameSite Cookie 等机制，而浏览器不需要用同源策略阻止写操作。

- 跨源读不仅能触发操作，还能窃取所有私有数据，且用户毫无感知。

- Web 早期就允许“嵌入”和“写”，而“读”的能力是后来才有的，为了应对“读”的风险，这才引入了 同源策略 + CORS 来限制。

- “嵌入”被允许是因为它本质上是“受限的读”。

| 资源类型                      | 能否被 JS 读取内容？         | 安全边界                                               |
| ------------------------- | -------------------- | -------------------------------------------------- |
| `<img>`                   | 只能读宽高、onload/onerror | 无法获取像素数据（除非 canvas + CORS）                         |
| `<script>`                | 无法读取脚本源码             | 只能执行，不能 inspect 内容                                 |
| `<link rel="stylesheet">` | 无法读取 CSS 规则（现代浏览器）   | 仅应用样式                                              |
| `<iframe>`                | 无法读取内容（除非同源）         | 受 `X-Frame-Options` / `Content-Security-Policy` 限制 |

“写比读更自由”不是漏洞，而是精心设计的安全边界。真正的防护责任在于：

- 浏览器：阻止未授权的“读”

- 服务器：防御未授权的“写”（通过 CSRF Token 等）

| 操作类型                 | 是否默认允许 | 原因                      |
| -------------------- | ------ | ----------------------- |
| **跨源写**（表单、链接）       | 是      | 不泄露数据，仅触发动作（需服务端防 CSRF） |
| **跨源嵌入**（img/script） | 是      | 内容不可被 JS 读取，仅渲染/执行      |
| **跨源读**（XHR/fetch）   | 否      | 会将敏感数据暴露给攻击者 JS         |

### 跨源资源共享

跨源资源共享（Cross-Origin Resource Sharing，CORS）是一个由一系列传输的 HTTP 标头组成的系统。这些 HTTP 标头决定浏览器是否阻止前端 JavaScript 代码获取跨源请求的响应。

同源安全策略默认阻止“跨源”获取资源。但是 CORS 给了 Web 服务器这样的权限，即服务器可以选择允许跨源请求访问到它们的资源。

跨源 HTTP 请求的一个例子：运行在 https://domain-a.com 的 JavaScript 代码使用 XMLHttpRequest 来发起一个到 https://domain-b.com/data.json 的请求。

跨源资源共享标准新增了一组 HTTP 标头字段，允许服务器声明哪些源站通过浏览器有权限访问哪些资源。另外，规范要求，对那些可能对服务器数据产生副作用的 HTTP 请求方法（特别是 GET 以外的 HTTP 请求，或者搭配某些 MIME 类型的 POST 请求），浏览器必须首先使用 OPTIONS 方法发起一个**预检请求**（preflight request），从而获知服务端是否允许该跨源请求。服务器确认允许之后，才发起实际的 HTTP 请求。在预检请求的返回中，服务器端也可以通知客户端，是否需要携带身份凭证（例如 Cookie 和 HTTP 认证相关数据）。

CORS 请求失败会产生错误，但是为了安全，在 JavaScript **代码层面无法获知**到底具体是哪里出了问题。你只能查看浏览器的控制台以得知具体是哪里出现了错误。

### 简单请求

某些请求不会触发 CORS 预检请求，称为“简单请求”。

其动机是，HTML 4.0 中的 `<form>` 元素（早于跨站 XMLHttpRequest 和 fetch）可以向任何来源提交简单请求，所以任何编写服务器的人一定已经在保护跨站请求伪造攻击（CSRF）。在这个假设下，服务器不必选择加入（通过响应预检请求）来接收任何看起来像表单提交的请求，因为 CSRF 的威胁并不比表单提交的威胁差。

若请求满足所有下述条件，则该请求可视为简单请求：

- 请求方法为 GET、HEAD、POST 之一。

- 除了被用户代理自动设置的标头字段，允许人为设置的字段为 Fetch 规范定义的对 CORS 安全的标头字段集合：Accept、Accept-Language、Content-Language、Content-Type、Range。

- Content-Type 标头所指定的媒体类型的值仅限于 text/plain、multipart/form-data、application/x-www-form-urlencoded 之一。

- 使用 XMLHttpRequest 对象发出的请求，在返回的 XMLHttpRequest.upload 对象属性上没有注册任何事件监听器。

- 请求中没有使用 ReadableStream 对象。

```
const fetchPromise = fetch("https://bar.other");

fetchPromise
  .then((response) => response.json())
  .then((data) => {
    console.log(data);
  });
```

浏览器发送给服务器的请求报文：

```
GET /resources/public-data/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Origin: https://foo.example
```

请求标头字段 Origin 表明该请求来源于 http://foo.example。

服务器响应：

```
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 00:23:53 GMT
Server: Apache/2
Access-Control-Allow-Origin: *
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml

[…XML Data…]
```

`Access-Control-Allow-Origin: *` 表明，该资源可以被任意外源访问。

### 预检请求

与简单请求不同，“需预检的请求”要求必须首先使用 OPTIONS 方法发起一个预检请求到服务器，以获知服务器是否允许该实际请求。"预检请求“的使用，可以避免跨域请求对服务器的用户数据产生未预期的影响。

```
const fetchPromise = fetch("https://bar.other/doc", {
  method: "POST",
  mode: "cors",
  headers: {
    "Content-Type": "text/xml",
    "X-PINGOTHER": "pingpong",
  },
  body: "<person><name>Arun</name></person>",
});

fetchPromise.then((response) => {
  console.log(response.status);
});
```

该请求包含了一个非标准的 HTTP X-PINGOTHER 请求标头，它不是 HTTP/1.1 的一部分。同时，该请求的 Content-Type 为 application/xml，且使用了自定义的请求标头。所以，该请求需要首先发起“预检请求”。

首先，浏览器发送给服务器预检请求：

```
OPTIONS /doc HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Origin: https://foo.example
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER, Content-Type
```

使用 OPTIONS 方法发送了预检请求。标头字段 Access-Control-Request-Method 告知服务器，实际请求将使用 POST 方法。标头字段 Access-Control-Request-Headers 告知服务器，实际请求将携带两个自定义请求标头字段：X-PINGOTHER 与 Content-Type。服务器据此决定，该实际请求是否被允许。

然后，服务器响应预检请求：

```
HTTP/1.1 204 No Content
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2
Access-Control-Allow-Origin: https://foo.example
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
Access-Control-Max-Age: 86400
Vary: Accept-Encoding, Origin
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
```

服务器将接受后续的实际请求方法（POST）和请求头（X-PINGOTHER）。响应携带了 Access-Control-Allow-Origin: https://foo.example，从而限制请求的源域。同时，携带的 Access-Control-Allow-Methods 表明服务器允许客户端使用 POST 和 GET 方法发起请求。标头字段 Access-Control-Allow-Headers 表明服务器允许请求中携带字段 X-PINGOTHER 与 Content-Type。标头字段 Access-Control-Max-Age 给定了该预检请求可供缓存的时间长短，86400 秒，也就是 24 小时。

最后，浏览器发送实际请求，服务器响应实际请求，类似于简单请求的两个步骤。

### 附带身份凭证的请求

一般而言，对于跨源 XMLHttpRequest 或 Fetch 请求，浏览器不会发送身份凭证信息。如果要发送凭证信息，需要设置 XMLHttpRequest 对象的某个特殊标志位，或在构造 Request 对象时设置。

```
const url = "https://bar.other/resources/credentialed-content/";

const request = new Request(url, { credentials: "include" });

const fetchPromise = fetch(request);
fetchPromise.then((response) => console.log(response));
```

Request 对象在构造时将 credentials 选项设置为 "include"，这是一个简单 GET 请求，所以浏览器不会对其发起预检请求。但是，浏览器会拒绝任何不带 Access-Control-Allow-Credentials: true 标头的响应，且不会把响应提供给调用的网页内容。

CORS 预检请求不能包含凭据。预检请求的响应必须通过指定 `Access-Control-Allow-Credentials: true` 来表明可以携带凭据进行实际的请求。

在响应附带身份凭证的请求时，服务器不能将 Access-Control-Allow-Origin、Access-Control-Allow-Headers、Access-Control-Allow-Methods、Access-Control-Expose-Headers 的值设为通配符（*），而应将其设置为特定的具体值，否则请求将会失败。

### CORS的请求头

- Origin 标头字段表明预检请求或实际跨源请求的源站。

- Access-Control-Request-Method 标头字段用于预检请求。其作用是，将实际请求所使用的 HTTP 方法告诉服务器。

- Access-Control-Request-Headers 标头字段用于预检请求。其作用是，将实际请求所携带的标头字段告诉服务器。

### CORS的响应头

- Access-Control-Allow-Origin 参数指定了单一的源，告诉浏览器允许该源访问资源。或者，对于不需要携带身份凭证的请求，服务器可以指定该字段的值为通配符“*”，表示允许来自任意源的请求。

- Access-Control-Expose-Headers 头将指定标头放入允许列表中，供浏览器的 JavaScript 代码获取。XMLHttpRequest 对象的 getResponseHeader() 方法只能拿到一些最基本的响应头，如果要访问其他头，则需要服务器设置本响应头。

- Access-Control-Max-Age 头指定了 preflight 请求的结果能够被缓存多久。

- Access-Control-Allow-Credentials 头指定了当浏览器的 credentials 设置为 true 时是否允许浏览器读取 response 的内容。当用在对 preflight 预检测请求的响应中时，它指定了实际的请求是否可以使用 credentials。

- Access-Control-Allow-Methods 标头字段指定了访问资源时允许使用的请求方法，用于预检请求的响应。

- Access-Control-Allow-Headers 标头字段用于预检请求的响应。其指明了实际请求中允许携带的标头字段。这个标头是服务器端对浏览器端 Access-Control-Request-Headers 标头的响应。

### CORS错误

**CORS disabled**

发送了一个需要使用 CORS 的请求，但在用户的浏览器中禁用了 CORS。发生这种情况时，用户需要在浏览器中重新打开 CORS。

**CORS header 'Access-Control-Allow-Origin' does not match 'xyz'**

发出请求的源不能与 Access-Control-Allow-Origin 标头允许的源相匹配。如果响应包含多个 Access-Control-Allow-Origin 标头，也会发生此错误。

**CORS header 'Access-Control-Allow-Origin' missing**

对 CORS 请求的响应缺少必需的 Access-Control-Allow-Origin 标头。

**CORS header 'Origin' cannot be added**

用户代理不能把所需的 Origin 标头添加到 HTTP 请求中，而所有的 CORS 请求必须有 Origin 标头。

**CORS preflight channel did not succeed**

CORS 请求需要预校验，但是不能执行预校验。可能是由于下列几种原因导致：

- 一个跨站请求在先前已经进行过预校验，进行重复的校验是不被允许的。请确保你的代码每次连接只进行一次预校验。 

- 预校验请求碰到了通常情况下不应该发生的网络问题。

**CORS request did not succeed**

使用 CORS 的 HTTP 请求失败，因为 HTTP 连接在网络或协议级别失败。该错误与 CORS 没有直接关系，而是某种基本的网络错误。

**CORS request external redirect not allowed**

服务器响应了 CORS 请求，并将 HTTP 重定向到与原始请求不同源的 URL 上，这在 CORS 请求期间是不允许的。

**CORS request not HTTP**

CORS 请求只能用于 HTTP 或 HTTPS URL 方案，但请求指定的 URL 可能是不同类型。这种情况经常发生在 URL 指定本地文件，例如使用了 file:/// 的 URL。

**Credential is not supported if the CORS header 'Access-Control-Allow-Origin' is '*'**

CORS 请求是在设置了凭证标志的情况下尝试的，但服务端使用通配符（"*"）配置 Access-Control-Allow-Origin 的值，这样是不允许使用凭证的。

**Did not find method in CORS header 'Access-Control-Allow-Methods'**

CORS 请求使用的 HTTP 方法不包含在响应的 Access-Control-Allow-Methods 标头指定的方法列表中。

**expected 'true' in CORS header 'Access-Control-Allow-Credentials'**

CORS 请求要求服务器允许使用凭据，但是服务器的 Access-Control-Allow-Credentials 标头的值并没有设置为 true。

**invalid token 'xyz' in CORS header 'Access-Control-Allow-Headers'**

服务器发送的对 CORS 请求的响应包含 Access-Control-Allow-Headers 标头，并且至少含有一个无效的标头名称。客户端用户代理在逗号分隔的值中找到由该标头提供的任何它无法识别的标头名称，都会发生此错误。

**invalid token 'xyz' in CORS header 'Access-Control-Allow-Methods'**

服务器发送的对 CORS 请求的响应包含 Access-Control-Allow-Methods 标头信息，并且含有至少一个无效的方法名称。客户端用户代理无法识别返回的值，则会发生此错误。

**missing token 'xyz' in CORS header 'Access-Control-Allow-Headers' from CORS preflight channel**

Access-Control-Allow-Headers 的值应该是逗号分隔的标头名称列表。

**Multiple CORS header 'Access-Control-Allow-Origin' not allowed**

服务器返回了多个 Access-Control-Allow-Origin 标头。这是不允许的。

### 在服务器中添加 CORS 支持

https://enable-cors.org/server.html

### CORS 问题诊断工具

https://httptoolkit.com/will-it-cors/source-url/

### 不带 CORS 的运行 Chrome 浏览器

https://alfilatov.com/posts/run-chrome-without-cors/

### HTML 属性：crossorigin

crossorigin 属性在 `<audio>`、`<img>`、`<link>`、`<script>` 和 `<video>` 元素中有效，它们提供对 CORS 的支持，定义该元素如何处理跨源请求，从而实现对该元素获取数据的 CORS 请求的配置。

crossorigin 属性可以取以下值：

- `anonymous`：请求使用了 CORS 标头，且证书标志被设置为 'same-origin'。没有通过 cookies、客户端 SSL 证书或 HTTP 认证交换用户凭据，除非目的地是同一来源。

- `use-credentials`：请求使用了 CORS 标头，且证书标志被设置为 'include'。总是包含用户凭据。

- 空值：与 anonymous 效果一致。（不合法的关键字或空字符串会视为 anonymous 关键字。）

`<img src="..." crossorigin="anonymous" />` 和 `<img src="..." />` 在跨域图片加载时有本质区别，主要体现在：是否触发 CORS 验证机制，以及后续能否通过 JavaScript 读取图片内容（如用于 Canvas）。

| 特性                 | `<img />`（无`crossorigin`） | `<img crossorigin="anonymous" />`    |
| ------------------ | ------------------------ | ------------------------------------ |
| **跨域请求类型**         | 普通跨域请求（不带 CORS 头）        | CORS 跨域请求（带`Origin`头）                |
| **是否受 CORS 策略约束**  | 否（浏览器允许加载）               | 是（需服务器响应 CORS 头）                     |
| **能否用于`<canvas>`** | 不能（会污染 canvas）           | 能（前提是服务器允许）                          |
| **错误处理**           | 加载失败仅`onerror`，无 CORS 错误 | 若 CORS 验证失败，触发`onerror`+ 控制台 CORS 错误 |

`<img src="..." />` 浏览器会正常加载跨域图片（同源策略允许“嵌入”），但该图片被标记为 “tainted”（污染），如果尝试用它绘制到 <canvas>，则将报错。因为 canvas 无法确定图片是否包含敏感内容（如用户私有图像），所以禁止读取像素数据。

```
const img = new Image();
img.src = 'https://other.com/image.jpg';
img.onload = () => {
  ctx.drawImage(img, 0, 0);
  ctx.getImageData(0, 0, 100, 100); // 抛出 SecurityError！
};
```

`<img src="..." crossorigin="anonymous" />` 会触发浏览器发起 CORS 请求。

必须用 crossorigin="anonymous" 的情况： 将跨域图片绘制到 Canvas 并导出（如海报生成、图像处理）；使用 WebGL/Three.js 加载跨域纹理；读取图片元数据（如 EXIF）。

值得注意的是，以下三种写法是等效的：

```
<img src="https://example.com/image.jpg" crossorigin />
<img src="https://example.com/image.jpg" crossorigin="" />
<img src="https://example.com/image.jpg" crossorigin="anonymous" />
```

### Vue 项目本地跨域解决方案

在 Vue 项目中，本地开发（localhost）时调用后端 API 遇到跨域问题是非常常见的场景。比如：

- 前端运行在 http://localhost:5173

- 后端 API 在 http://localhost:3000

浏览器因同源策略拦截请求，报错：

```
Access to fetch at 'http://localhost:3000/api' from origin 'http://localhost:5173' has been blocked by CORS policy.
```

**使用 Vite（Vue 3 默认）**

修改 vite.config.js

```
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  server: {
    proxy: {
      // 将所有以 /api 开头的请求代理到后端
      '/api': {
        target: 'http://localhost:3000', // 后端地址
        changeOrigin: true,              // 改变 origin 为目标地址（重要！）
        rewrite: (path) => path.replace(/^\/api/, '') // 可选：重写路径
      }
    }
  }
})
```

前端代码中请求 `/api/xxx`

```
// src/api/user.js
fetch('/api/users') // 实际被代理到 http://localhost:3000/users
```

**使用 Vue CLI（基于 Webpack）**

修改 vue.config.js

```
// vue.config.js
module.exports = {
  devServer: {
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true,
        pathRewrite: {
          '^/api': '' // 重写路径，去掉 /api 前缀
        }
      }
    }
  }
}
```

前端请求同样使用 `/api/xxx`

```
axios.get('/api/profile')
```