---
layout: post
title:  "RN项目一加手机白屏问题排查"
date:   2025-08-12 19:41:00 +0800
---

* 目录
{:toc #markdown-toc}

### 1 问题排查

（1）问题表现为首页白屏（白屏是首页，不是开屏页）。

（2）问题定位到是原生首页的WebView加载H5首页时报错。

![img](../../../assets/oneplus-phone-white-screen/1.png)

（3）问题只出现在一加手机 OnePlus9上，另一台一加手机 OnePlus Ace2Pro无此问题。两台手机的系统配置差异如下：

- OnePlus9：Android11, ColorOS11.2, WebView83.0.4103.106

- OnePlus Ace2Pro：Android14, ColorOS14.0, WebView117.0.4103.106

（4）将RN项目的原生首页所加载的链接替换为百度首页（https://www.baidu.com/），可以成功加载。

（5）一加手机 OnePlus9 的内置浏览器可以打开H5首页。

（6）与RN项目的react-native-webview依赖无关，RN项目的此项依赖与社区版有区别（集成了腾讯的QbSDK），但是调整为社区版后，问题仍然存在。

（7）经了解，"Uncaught SyntaxError: Unexpected token '='"可能的原因是，H5使用了ES6语法，而WebView不支持。但是，自Android6.0始，WebView即支持ES6语法。

（8）初步结论：RN项目的WebView可以加载百度首页，加载不了H5首页，说明WebView没问题，H5有问题。但是，H5首页在手机内置浏览器上可以成功加载，说明H5没问题，WebView有问题。

（9）在本地运行H5项目，定位到"Uncaught SyntaxError: Unexpected token '='"发生的确切位置：

```
[INFO:CONSOLE(211449)] "Uncaught SyntaxError: Unexpected token '='", source: http://192.168.140.92:4200/vendor.js (211449)
```

对应位置的代码如下：

```
// vendor.js~line#211449
partialText = request.responseType === 'text' ? (partialText ?? '') + (decoder ??= new TextDecoder()).decode(value, {
```

其中，运算符 '??=' 是在ECMAScript 2021（ES12）中引入的一个新特性。

据了解，ECMAScript 2021（ES12），发布于2021年6月，而一加手机OnePlus9的WebView版本83.0.4103.106则发布于2020年6月。

（10）基于以上，问题的解决可从两个方面的着手：

- 调整H5项目，使其兼容不支持ES12的浏览器；

- 调整RN项目，使其WebView总是支持ES12；

### 2 问题解决

（1）考虑到难易程度和时间节点，采用调整H5项目的方式。

（2）据了解，决定Angular项目构建输出所基于的ES版本的参数位于tsconfig.json的compilerOptions配置节：

```
"compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "lib": [
      "es2022",
      "dom"
    ],
}
```

几个配置参数的含义见于[<u><span class="15"><font face="Calibri">compiler-options</font></span></u>](https://www.tslang.cn/docs/handbook/compiler-options.html)，节其要点如下：

- target：指定ECMAScript目标版本（即构建输出所基于的版本）；

- module：指定生成哪个模块系统代码（经验证，其与target存在着关联性）；

- lib：编译过程中需要引入的库文件的列表；

（3）尝试调整上述几个参数，并验证：

- target: "ES6" / module: "ES2022" / lib: ["ES2022", "dom"]

  此种情况下，PC浏览器（Chrome）表现正常，但手机上的WebView仍然报错（"Uncaught SyntaxError: Unexpected token '='"）。

  把 "ES2022" 调整为 "ES2020"，结果相同。

- target: "ES6" / module: "ES2015" / lib: ["ES2015", "dom"]

  编译报错：ES2015不能支持动态导入。

```
Error: src/app/app.routes.ts:25:7 - error TS1323: Dynamic imports are only supported when the '--module' flag is set to 'es2020', 'es2022', 'esnext', 'commonjs', 'amd', 'system', 'umd', 'node16', or 'nodenext'.

25       import(
         ~~~~~~~
26         /* webpackChunkName: 'home' */
   ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
27         './page/daily-wallpaper/daily-wallpaper.component'
   ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
28       ).then((mod) => mod.DailyWallpaperComponent),
   ~~~~~~~
```

（4）基于可操作性层面的考虑，采用后一种配置（即"ES2015"），对代码中的动态导入做处理（只存在于如下两个文件）：

- app.routes.ts：将动态导入的组件调整为一般导入形式。

- app.component.ts中的动态导入逻辑注释（**待进一步处理**）。

（5）做上述处理后，仍未解决问题，手机上的WebView仍然报错（"Uncaught SyntaxError: Unexpected token '='"）。

```
// vendor.js~line#211449
partialText = request.responseType === 'text' ? (partialText ?? '') + (decoder ??= new TextDecoder()).decode(value, {
```

进一步定位报错代码的位置，即 '??=' 运算符出现的位置，系项目依赖库中的代码，其代码文件位于：.\node_modules\@angular\common\esm2022\http\src\fetch.mjs

![img](../../../assets/oneplus-phone-white-screen/2.png)

问题未解决的原因是：tsconfig.json中的配置仅对src目录下的代码生效，而对依赖库中的代码未起作用。

（6）经了解，构建过程中，实现高版本ES特性的代码转换到低版本ES的形式，产生作用的是babel。babel作为一个转换规则嵌入到webpack的构建过程中。Angular项目对webpack构建过程做自定义配置，基于angular.json文件的如下配置节：

```
project.{$project-name}.architecture.build.options.customWebpackConfig
```

H5项目已指定其为：src/custom-webpack.config.js

（7）在custom-webpack.config.js中指定babel规则：

```
module.exports = {
  devtool: "source-map",
  module: {
    rules: [
      {
        test: /\.m?js$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env'],
            sourceType: 'unambiguous',
            cacheDirectory: true
          }
        }
      },
    ]
  },
  ...
}
```

通过执行npm run build命令进行构建，再将构建结果部署到本地的nginx服务，

![img](../../../assets/oneplus-phone-white-screen/3.png)

然后，在RN项目上访问本地服务。经验证，规则生效，一加手机OnePlus9上安装的App可以成功加载首页Web，不再出现白屏。
