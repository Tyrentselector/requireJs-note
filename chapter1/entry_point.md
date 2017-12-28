# data-main 入口

data-main 是一个很特殊的属性 RequireJs 会自动检测它并加载指定脚本：
```
<!--RequireJs 加载 scripts/main.js 并且注入一个新标签（使用 async 属性）-->
<script data-main="scripts/main" src="scripts/require.js"></script>
```

通常我们使用 data-main 所指定的脚本来设定[配置选项]()并且加载第一个模块。RequireJs 为 **data-main 模块** 生成的**script 标签**含有[async 属性](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#attr-async)。**这意味着我们无法确定 data-main 所指向的脚本相较于本页面中其它脚本优先完成加载或执行。**

例如，下面这例子会随机失败：
```
<script data-main="scripts/main" src="scripts/require.js"></script>
<script src="scripts/other.js"></script>
```

```
// contents of main.js:
require.config({
    paths: {
        foo: 'libs/foo-1.1.3'
    }
});
```

```
// contents of other.js:

// 该脚本可能在 main.js 中 requirejs.config() 之前调用。
// 如果 本脚本在 mian.js 前调用, RequireJs 会尝试区加载
// 'scripts/foo.js' 而不是 'scripts/libs/foo-1.1.3.js'
require(['foo'], function(foo) {

});
```

如果想在 HTML 页面中调用 **require()**，最好不要使用 **data-main** 属性。**data-main** 只适用于没有主入口的页面。对于那些想要使用内联调用 **require()** 的页面，最好是嵌套在加载配置的 **require()** 函数内部：
```
<script src="scripts/require.js"></script>
<script>
require(['scripts/config'], function() {
    // 配置已经加载, 确保其他可以依赖 config.js 的模块正确加载依赖
    require(['foo'], function(foo) {

    });
});
</script>
```