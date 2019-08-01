---
title: webpck4分享
date: 2019-01-15 21:20:32
tags:
---
# vue-cli项目webpack3升级到webapck4分享
webpack4出来也挺久了，那么webpack4相比webpack3给我们的项目带来了那些变化：

1.速度大幅度提升，官方在发布的时候也曾表示，其编译速度提升了 60% ~ 98%。

2.添加了mode模式，--development开发模式，--production编译模式。

3.开箱即用 WebAssembly，webpack4提供了wasm的支持，现在可以引入和导出任何一个 Webassembly 的模块，也可以写一个loader来引入C++、C和Rust。
 (这一点暂时还没有了解)

4.全新的插件系统(比如一些插件的废除，新增) 

我们通过用webpack3的构建的vue项目升级来体验一下。首先我们vue-cli2来构建一个vue的项目，这一步相信大家都会了，如果没有构建过大家可以上网上看看列子，很简单的。

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
webpack4废弃了这个方法了，取而代之的是使用iptimization来打包公共的引用模块。我们项目中找到commonsChunkPlugin方法，这个插件引用主要是在
webpack.prod.conf.js这个文件下面，只有在打包的时候用到，我们删除掉commonsChunkPlugin的使用，在里面添加上
```
new webpack.optimize.SplitChunksPlugin({
    chunks: "all",
    minSize: 30000,
    minChunks: 1,
    maxAsyncRequests: 5,
    maxInitialRequests: 3,
    name: true,
    cacheGroups: {
        default: {
            minChunks: 2,
            priority: -20,
            reuseExistingChunk: true,
        },
        vendors: {
            test: /[\\/]node_modules[\\/]/,
            priority: -10
        }
    }
})
```
接着替换extract-text-webpack-plugin这个插件成mini-css-extract-plugin，因为extract-text-webpack-plugin已经被废除了在webpack4中。
我们在webpack.prod.conf.js中去除extractTextPlugin,添加MiniCssExtractPlugin.这个插件作用是在webpack4中提取css。
```
 - new ExtractTextPlugin({
      - filename: utils.assetsPath('css/[name].[contenthash].css'),
      // Setting the following option to `false` will not extract CSS from codesplit chunks.
      // Their CSS will instead be inserted dynamically with style-loader when the codesplit chunk has been loaded by webpack.
      // It's currently set to `true` because we are seeing that sourcemaps are included in the codesplit bundle as well when it's `false`, 
      // increasing file size: https://github.com/vuejs-templates/webpack/issues/1110
      - allChunks: true,
    - }),
+ new MiniCssExtractPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css')
    })
```
替换完webpack.prod.conf文件后，我们在找到utils.js里面，这个js里面也有使用到。
```
// build/utils.js

var MiniCssExtractPlugin = require("mini-css-extract-plugin");

if (options.extract) {
    -return ExtractTextPlugin.extract({
        -use: loaders,
        -fallback: 'vue-style-loader'
      -})
    + return [MiniCssExtractPlugin.loader].concat(loaders)
} else {
    return ['vue-style-loader'].concat(loaders)
}

```
替换完以上这些内容，我们升级就完成了，重新打包后，你会发现打包速度有明显的提高。
如果大家有兴趣可以在升级babel,因为这边babel还是有6.x的版本。

