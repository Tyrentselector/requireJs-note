# 3 配置选项

在顶层 HTML 页面（或没有定义模块的顶级脚本文件）使用 **require()** 时，配置对象可以作为第一选项进行传参：
```
<script src="scripts/require.js"></script>
<script>
  require.config({
    baseUrl: "/another/path",
    paths: {
        "some": "some/v1.0"
    },
    waitSeconds: 15
  });
  require( ["some/module", "my/module", "a.js", "b.js"],
    function(someModule,    myModule) {
        // 在上面那所有依赖加载完成后会调用这个函数
        // 注意这个函数可以在页面完成加载前就会被执行
        // 这个回调函数不是必须的。
    }
  );
</script>
```

我们同样可以在 [data-main 入口](../chapter1/entry_point.md) 中调用 **require.config**，不过请注意 **data-main 脚本**是**异步加载**的。避免再出现其它入口，这些入口脚本会误以为 **data-main** 文件的加载和 data-main 文件中的 require.config 函数的执行会优先于这些脚本的加载。

我们也可以在 require.js 加载前定义一个全局对象 **require**，它包含一些自动应用的值：
```
<script>
    var require = {
        deps: ["some/module1", "my/module2", "a.js", "b.js"],
        callback: function(module1, module2) {
        // 在上面那所有依赖加载完成后会调用这个函数
        // 注意这个函数可以在页面完成加载前就会被执行
        // 这个回调函数不是必须的。
        }
    };
</script>
<script src="scripts/require.js"></script>
```