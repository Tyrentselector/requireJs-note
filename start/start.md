# 1. 快速入门

## 1.1 获取 RequireJs

从[官网](http://requirejs.org/docs/release/2.3.5/comments/require.js)下载

## 1.2 使用 RequireJs
1. 目录结构

```
project-directory/
    project.html
    scripts/
        main.js
        require.js
    helper/
        util.js
```

2. 在 index.html 设置入口

```
<!DOCTYPE html>
<html>
    <head>
        <title>My Sample Project</title>
        <!-- data-main attribute tells require.js to load
             scripts/main.js after require.js loads. -->
    </head>
    <body>
        <h1>My Sample Project</h1>
    </body>
    <script data-main="js/main" src="scripts/require.js"></script>
</html>
```

为了防止 RequireJs 脚本加载阻塞页面渲染，可以将入口标签放置在 body 后面。或者使用 [async](http://www.growingwiththeweb.com/2014/02/async-vs-defer-attributes.html) 属性 。requirejs 加载完成后会根据入口 script 标签中的 *data-main* 属性加载入口文件并自动运行加载依赖。

3. 在 main.js 中你可以使用 **requirejs()** 函数加载任何你需要执行的脚本。因为 data-main 属性指定的脚本是异步加载的，所以要确保唯一入口。
```
requirejs(["helper/util"], function(util) {
    // 当 js/helper/util.js 加载完成后这个函数会自动调用
    // 如果 util.js 中调用了 defined() 函数，该函数会在 util.js 的依赖文件
    // 全部加载完成后执行, 同时参数 util 将包含
    // helper/util" 模块的值
});
```
