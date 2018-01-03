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
> 注意：最好使用 ```var require = {}``` 不要使用 ```window.require = {}```，后者在 IE 中的行为有偏差。

这里有一些[从主模块中分离配置的模板](https://github.com/requirejs/requirejs/wiki/Patterns-for-separating-config-from-the-main-module)。

配置参数：

**baseUrl**：查找所有模块的根路径。对于上面的例子，**my/module** script 标签的 src 属性值为 "/another/path/my/module.js"。但是当加载 .js 后缀文件时不会使用 **baseUrl**。[这些字符串](../chapter1/load_files.md)按原样使用，所以 **a.js** 和 **b.js** 会从包含以上片段的 HTML 页面的同级目录进行加载。

如果在配置中没有明确指明 **baseUrl** ，那么它的默认值将会自动被设定为加载 require.js 的 HTML 页面所在目录。如果使用了 **data-main** 属性，那么它的值就是 **baseUrl** 的值。

baseUrl 可以与加载 require.js 的页面的 URL 不同域。 RequireJs 可以跨域加载脚本。唯一的限制是通过 text! 插件加载文本内容：这些文本的路径应该同页面处于同一域中。优化工具中将会内置 text! 插件资源，所以在使用优化工具后我们可以使用 text! 插件从其它域引用的资源。

**paths**：路径映射用于不能直接在 baseUrl 目录下直接找到的模块名。路径设置默认是相对于 baseUrl 的，除非路径设置以  "/" 或含有协议的 URL 例如( http:)。使用以上样例配置，"some/module" script 标签的 src="/another/path/some/v1.0/module.js"。用于模块名的路径不应该包含扩展名，因为路径映射应该是一个目录。当将一个模块名映射为路径时，路径映射代码会自动添加 .js 后缀。如果使用了[require.toUrl()](../chapter1/define_module.md#toUrl)，它会增加合适的扩展名。

在浏览器中运行时，可以指定[备用路径]()，在尝试从 CDN 中加载文件失败后，会尝试从备用路径加载文件。

**bundles**：配置多个模块 ID，可以通过这些模块 ID 找到另一个脚本：
```
requirejs.config({
    bundles: {
        'primary': ['main', 'util', 'text', 'text!template.html'],
        'secondary': ['text!secondary.html']
    }
});

require(['util', 'text'], function(util, text) {
    // 在这个脚本中模块 ID “primary” 已经被加载，
    // 并且 primary 模块包含 'util' 和 'text' 模块
});
```

以上配置：通过加载 ID 为 'primary' 的模块能够找到 'main', 'util', 'text' 和 'text!template.html' 等模块。通过加载 ID 为 'secondary' 的模块可以找到模块 'text!secondary.html'。

它用于设置如何在一个包含多个定义模块的脚本中找到一个模块。并不会自动把这些模块捆绑到这个包模块 ID 上。包模块 ID 仅用于定位这个模块包。

这可能与路径配置很相似，但是这个配置比较冗长，同时在路径配置路线不允许包含加载器插件资源 ID，因为路径配置的值应该是路径片段，而非 ID。

优化和压缩我们的脚本至一个单个的 js 文件中被认为是前端最佳实践，同时这样对 HTTP 请求是十分友好的——它可以减少 HTTP 请求，对于移动端来说这十分重要。然而随着项目的发展我们会发现我们优化的文件会越来越大。这是当前移动端所面临的窘境。这样不但会加重网络负担，同时也会加重浏览器负担。

**** 允许我们把模块整合成一个优化过的文件包。当 app 需要包中的一个模块时，RequireJs 会加载相应的模块。例如，我们可能有一个网站中所有复杂页面都需要的包：'homepage', 'articleslist', 'projects' 等和一个叫做 **shared** 的**包**含有网站容器和一些公共方法，同时这个 **shared** 包在其他页面中也被使用。

通过 requirejs 配置对象我们定义 **bundles**：
```
requirejs.config({
  bundles: {
    'shared': ['shared/main', 'shared/util', 'shared/site', 'text!shared/templates/site.html'],
    'homepage': ['homepage/home', 'text!homepage/templates/home.html'],
    'articleslist': ['articleslist/articles', 'text!articleslist/templates/articles.html']
    // etc...
  }
});
```

这是一个非常简单的设置，但是它告诉 RequireJs 可以在那个包中找到那个模块。当网站运行时 RequireJs 并尝试加载一个模块时，它首先会查看这个配置找出从那个包中加载这模块。如果这个包还没有加载 RequireJs 会去加载它。这意味这我们的网站只需加载最低限度的包。

如果我们有一个单一入口页面的 app，我们想要充分利用这一点在运行时加载模块。下面这个例子 'site' 模块包含网站范围菜单，并且在点击相关内容时将会响应式加载相关页面：
```
define(['site'], function(site) {
  // 当点击导航至 articleslist 在运行时加载这个模块:
  require(['articles'], function(articles) {
    // RequireJS 现在已经从 articleslist 包中加载了 articles 模块
  });
});
```

自 RequireJs 2.2.0 起，优化器可以生成包配置并将它插入到顶级 requirejs.config() 中调用。详见 [bundlesConfigOutFile ](https://github.com/requirejs/r.js/blob/98a9949480d68a781c8d6fc4ce0a07c16a2c8a2a/build/example.build.js#L641)。

