# 加载 js 文件

RequireJs 采用与传统方式不同的脚本加载方式。它能更快更好的加载脚本，同时 RequireJs 主要目标是支持模块化编码。RequireJs 支持使用模块 ID 替代含有 URls 的 script 标签。

RequireJs 相对于 [baseUrl]() 进行模块加载。通常 **baseUrl** 设定为 **data-main** 属性值的脚本的同级目录。RequireJs 使用 **data-main** 属性检测并加载脚本。
```
<!--baseUrl 被设定为 'scripts' 目录-->
<!--baseUrl 被设定为 **scripts** 目录, 并且加载一个模块 ID 为 'main' 脚本文件-->
<script data-main="scripts/main.js" src="scripts/require.js"></script>
```

同时也可以通过 [RequireJs config]() 手动设置 **baseUrl**,如果没有明确指定配置同时没有使用 **data-main** 属性，那么 **baseUrl** 会被设定为运行 RequireJs 的 HTML 页面所在目录。

RequireJs 假设所有依赖都是 scripts，所以不用为依赖的模块ID添加 **.js** 后缀，RequireJs 会在将模块 ID 转换为路径时自动添加后缀。使用 [paths 配置]() 你可以设定一组脚本的位置。相较于传统的 ```<script>``` 标签，这些能力能让我们能够使用更短的字符串来加载脚本。

有时我们想要直接引用一个脚本并且无法 **baseUrl + paths** 的规则来找到它。如果模块拥有以下任一特征，ID 将不会通过 **baseUrl + paths** 配置，这些模块将会作为常规 URL 模块处理：
    + 以 ".js" 结尾
    + 以 "/" 开头
    + 包含 Url 协议，例如 "http" 或者 "https"

通常最好使用 baseUrl 和 **paths** 的方式来设置模块路径。通常这样做能够给我们让我们在重命名及变更目录时更灵活。

同样为了避免过长的配置，最好避免文件夹深层嵌套。如果希望将第三方类库与我们的源代码进行分离，最好使用类似以下的目录结构：
```
www/
    index.html
    js/
    app/
        sub.js
    lib/
        jquery.js
        canvas.js
    app.js
    require.js
```

在 **index.html** 中：
```
<script data-main="js/app.js" src="js/require.js"></script>
```

在 **app.js** 中：
```
requirejs.config({
    //默认从 js/lib 中加载模块
    baseUrl: 'js/lib',
    // 如果模块 ID 以 "app" 开头,
    // 从 js/app 目录下加载该脚本。
    // 路径的配置相对于 baseUrl, 并且
    // 不要加 '.js' 扩展名因为路径配置
    // 应该是一个目录
    paths: {
        app: '../app'
    }
});

// 应用主逻辑入口
requirejs(['jquery', 'canvas', 'app/sub'],
function   ($,        canvas,   sub) {
    // jQuery, canvas 和 app/sub 模块
    // 现在已经可以使用啦。
});
```

注意示例部分第三方类库 JQuery 文件名中不包含版本号。如果你想跟踪 JQuery 版本，推荐使用另一个独立的文件保存版本号，或者使用类似 [volo](https://github.com/volojs/volo) 的工具。它会在 **package.json** 中标记版本信息同时保证文件在磁盘中的名称为 "jquery.js"。

理论上我们加载的所有脚本都是通过调用 [define()]() 定义的模块。然而我们可能需要使用一些**“浏览器全局脚本”**，这些脚本不能通过 **defined()** 传递它们的依赖。对于这些脚本我们可以使用 [shim config]()。来正确表示它们的依赖。

如果我们没有表述依赖，我们很可能会遇到加载错误的，因为 RequireJs 会无序异步地加载脚本。
