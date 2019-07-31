---
title: webpck4分享
date: 2019-01-15 21:20:32
tags:
---
# vue-cli项目webpack3升级到webapck4分享
webpack4出来也挺久了，那么webpack4相比webpack3给我们的项目带来了那些变化，我们通过对webpack3的vue项目升级来了解一下。

首先我们vue-cli2来构建一个vue的项目，这一步相信大家都会了，如果没有构建过大家可以上网上看看列子，很简单的。

可以看到生成完的项目package.json里面webpack的相关库版本如下：
```
"vue-loader": "^13.3.0",
"vue-style-loader": "^3.0.1",
"vue-template-compiler": "^2.5.2",
"webpack": "^3.6.0",
"webpack-bundle-analyzer": "^2.9.0",
"webpack-dev-server": "^2.9.1",
"webpack-merge": "^4.1.0"
```
1.首先我们webpack,webpack-dev-server升级到最新的版本。升级完这两个第三库后运行npm run dev 这个时候会报如下错误：
```
Error: Cannot find module 'webpack-cli/bin/config-yargs'
```
这是由于项目中缺少webpack-cli,由于我们是全局安装webpack-cli2.9版本，所以需要在当前项目下安装最新的webpack-cli.重新运行，结果发现又报错了
```
ace/webpack-update/node_modules/html-webpack-plugin/lib/compiler.js:81
        var outputName = compilation.mainTemplate.applyPluginsWaterfall('asset-path', outputOptions.filename, {
                                                  ^

TypeError: compilation.mainTemplate.applyPluginsWaterfall is not a function
    at /Users/lincoln/JavaScript_WorkSpace/webpack-update/node_modules/html-webpack-plugin/lib/compiler.js:81:51
```
这是由于版本兼容性的问题,我们把html-webpack-plugin升级到最新版本。

更新完webpack对象的插件后我们来更新vue-loader,eslint-loader插件，如果不更新也是会造成版本兼容问题。升级完这两个插件后，运行后，发现报
```
vue-loader was used without the corresponding plugin. Make sure to include VueLoaderPlugin in your webpack config.
You may use special comments to disable some warnings.
Use // eslint-disable-next-line to ignore the next line.
Use /* eslint-disable */ to ignore all warnings in a file.
 error  in ./src/App.vue?vue&type=style&index=0&lang=css&

Module parse failed: Unexpected character '#' (15:0)
File was processed with these loaders:
 * ./node_modules/vue-loader/lib/index.js
 * ./node_modules/eslint-loader/index.js
You may need an additional loader to handle the result of these loaders.
|
|
> #app {
|   font-family: 'Avenir', Helvetica, Arial, sans-serif;
|   -webkit-font-smoothing: antialiased;

 @ ./src/App.vue 4:0-63
 @ ./src/main.js
```
从官网得知，需要加入VueLoaderPlugin这个插件。
接着我们重新运行，报了
```
Error: webpack.optimize.CommonsChunkPlugin has been removed, please use config.optimization.splitChunks instead.
```
webpack4废弃了这个方法了，取而代之的是使用iptimization来打包公共的引用模块。
接着替换extract-text-webpack-plugin这个插件成mini-css-extract-plugin，因为extract-text-webpack-plugin已经被废除了在webpack4中。

到这里我们升级就完成了，如果大家有兴趣可以在升级babel,因为这边babel还是有6.x的版本。

