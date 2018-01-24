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
<a id="configBlock"></a>
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

<a id="paths"></a>
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

<a id="configShim"></a>
**shim**：为没有使用 **define()** 声明依赖和模块值的传统“浏览器全局”脚本配置依赖、exports 和自定义初始化值。

下面这个例子，需要 RequireJs 2.1.0+，并假定 backbone.js, underscore.js 和 jquery.js 安装在 baseUrl 目录中。如果它们在其他目录中，那么我们就需要为他们设定路径配置：
```
requirejs.config({
    // 注意：shim 仅能使用在那些没有使用 AMD 规范声明的脚本上，
    // 这些脚本没有使用 define()。如果对 AMD 脚本使用 shim 配置
    // 它将会出现错误，exports 和 init 配置不会被触发，同时 deps
    // 配置将会变得混乱。
    shim: {
        'backbone': {
            // 这些依赖脚本应该在 backbone.js 加载前完成加载
            deps: ['underscore', 'jquery'],
            // 一旦加载完成, 使用全局 'Backbone' 作为模块值
            exports: 'Backbone'
        },
        'underscore': {
            exports: '_'
        },
        'foo': {
            deps: ['bar'],
            exports: 'Foo',
            init: function (bar) {
                // 在这个函数中我们可以调用类库支持的 noConflict 方法，并做一些其它的初始化   操作。
                // 然而, 这些类库的插件可能依然期望获取这个类库的一个全局变量
                // 在这个函数中 "this" 指向全局对象。依赖将会作为参数传入
                // 如果这个函数返回一个值，那么这个值将会替代通过 'exports' 字符串
                // 找到的对象作为模块的导出值。
                // 注意：jQuery 注册器是一个通过 define() 定义的 AMD 模块。
                // 所以这个配置对 jQuery 是不起作用的，在注意事项部分可以找到对应的处理方法。
                return this.Foo.noConflict();
            }
        }
    }
});

// 之后，在一个单独的文件中调用使用 AMD 规范定义的 'MyModel.js' 模块，
// 并指定 'backbone' 作为它的一个依赖。RequireJs 将会根据 shim 配置
// 正确的加载 'backbone' 并为这个模块提供一个本地引用。全局 Backbone
// 依旧存在于这个页面中。
define(['backbone'], function (Backbone) {
  return Backbone.Model.extend({});
});
```

对于 jQuery 或 Backbone 这类无需导出任何模块值的模块，shim 配置可以是一个依赖数组：
```
requirejs.config({
    shim: {
        'jquery.colorize': ['jquery'],
        'jquery.scroll': ['jquery'],
        'backbone.layoutmanager': ['backbone']
    }
});
```

注意如果希望在 IE 中获取 404 加载检查以便使用**备用路径**或**错误回调**，那么**exports**选项应该被给定一个字符串类型的值，以便加载器用于检测脚本是否真的被加载了（**init** 配置项中的返回值不能用于 ```enforceDefine``` 检测）：
```
requirejs.config({
    shim: {
        'jquery.colorize': {
            deps: ['jquery'],
            exports: 'jQuery.fn.colorize'
        },
        'jquery.scroll': {
            deps: ['jquery'],
            exports: 'jQuery.fn.scroll'
        },
        'backbone.layoutmanager': {
            deps: ['backbone']
            exports: 'Backbone.LayoutManager'
        }
    }
});
```

**关于 "shim" 配置的重要提示：**
    <a style="color: red;">不明白为什么不能在构建时同时使用CDN + shim</a>
1. shim 配置仅能用于设置代码关系。只是配置 shim 并不会触发模块加载。加载模块还是要使用 require/define。
2. 仅使用其他 "shim" 模块作为 "shim" 脚本的依赖项。一个模块如果它符合AMD规范，没有其他依赖，同时在调用 define() 后依旧创建全局变量 （例如 JQuery 和 Backbone）的类库也可以作为 "shim" 脚本的依赖。否则，如果使用一个 AMD 模块作为一个 "shim" 模块的依赖，在构建后，会引发一个错误。终极解决方案是将所有模块升级为 AMD 模块。
3. 自 RequireJs 2.1.11起，优化器包含一个 [wrapShim build]() 配置项，在build时它会尝试自动为 "shim" 脚本包裹 define() 外壳。这会改变 "shim" 的依赖范围，所以这个方法不是总是有效的。但是，例如，一个 "shim" 脚本依赖符于合 AMD 规范的 Backbone，它是起作用的。
4. 对于 AMD 模块 init 函数不会被调用。例如，我们无法使用 shim init 函数来调用 JQuery 的 noConflict。见 [ Mapping Modules]() 使用 JQuery 的 noConflict。

**对于优化器 "shim" 配置的重要事项：**
1. 我们应该使用 **mainConfigFile** 构建选项指定包含 **shim** 配置的文件。否则优化器无法知道 **shim** 配置。其他选项会复制构建配置中的 **shim** 配置。
2. 如果我们要在构建时对代码进行混淆，**不要**设定混淆 ```toplevel``` 选项为 **true**，或使用 cli 时**不要**传入 ```-mt```。这个选项会破坏 **shim** 用于查找 **exports** 的全局变量名。

**maps**：这个配置对于大型应用是很有用的，这种大型应用可能包含两套模块，同时这两个模块需要两个不同的版本的 'foo'，但它们之间任然需要彼此协作。

通过[多版本支持]()无法实现上述功能。此外[paths 配置](#paths)只适用于为模块 ID 设置根路径，不是为了映射一个模块 ID 到另一个模块中存在的。
```
requirejs.config({
    map: {
        'some/newmodule': {
            'foo': 'foo1.2'
        },
        'some/oldmodule': {
            'foo': 'foo1.0'
        }
    }
});
```

模块布局结构如下：
```
foo1.0.js
foo1.2.js
some/
    newmodule.js
    oldmodule.js
```

当'some/newmodule' 调用 `require('foo')` 时它将会去请求 **foo1.2.js** 文件，当'some/oldmodule' 调用 `require('foo')` 时它将会去请求 **foo1.0.js** 文件。
该特性只适用于 AMD 模块，同时该模块为 **defined() 定义的一个匿名函数**。同时，map 配置只能使用**绝对模块ID**。相对 ID（例如：'../some/thing'）不起作用。

同时也支持'*'映射值，它意味着：加载模块时，都使用这个 map 配置。如果含有多个map配置，那么其它配置会优先于'*'：
```
requirejs.config({
    map: {
        '*': {
            'foo': 'foo1.2'
        },
        'some/oldmodule': {
            'foo': 'foo1.0'
        }
    }
});
```
除了 'some/oldmodule' 模块外的所有模块在请求 'foo' 时实际请求的是 'foo1.2'。对于 'some/oldmodule' 模块在请求 'foo' 时请求的是 'foo1.0'。

<div style="color:red;">学习完optimizer后再进行翻译</div>
在构建时，map 配置需要提供给优化器。

**config**：通常用于将配置信息传递给模块。这些配置信息通常被当作应用的一部分，同时需要一种方式将它们向下传递至一个模块。在 RequireJs 中可以通过 requirejs.config() 中的 **config** 配置项完成。模块可以读取这些配置，通过引用特殊依赖 **module** 同时调用 **module.config()** 例如：
```
requirejs.config({
    config: {
        'bar': {
            size: 'large'
        },
        'baz': {
            color: 'blue'
        }
    }
});

//bar.js, which uses simplified CJS wrapping:
define(function (require, exports, module) {
    //Will be the value 'large'
    var size = module.config().size;
});

//baz.js which uses a dependency array,
//it asks for the special module ID, 'module':
define(['module'], function (module) {
    //Will be the value 'blue'
    var color = module.config().color;
});
```

传递配置到 [package]() 时，应该将 package 中的主模块作为目标，而不是 package ID：
```
requirejs.config({
    // 向 pixie 包的主模块传递一个 API 键
    config: {
        'pixie/index': {
            apiKey: 'XJKDLNS'
        }
    },
    // 配置 pixie 包，它的主模块是 pixie 文件夹中的 index.js 文件
    packages: [
        {
            name: 'pixie',
            main: 'index'
        }
    ]
});
```

**packages**：从 CommonJs 包中加载安装模块。详见 [packages 主题]()。

**nodeIdCompat**：NodeJs 认为模块 ID ```example.js``` 和 ```example``` 是一样的。在 RequireJs 中默认认为这是两个不同的模块 ID。如果我们使用通过 npm 安装的模块，那么我们可能需要设定这项配置的值为 **true** 来避免解析问题。该选项仅用来处理 **.js** 后缀的差异。

**waitSecond**：放弃脚本加载前的等待时间。设定为 **0** 禁用超时。默认为7s。

**context**：在一个页面中允许 RequireJs 加载多个版本的模块，只要每个顶级的 require 调用指定唯一的上下文字符串。详见[多版本支持]()。

**deps**：一个待加载依赖数组。当 require.js 加载前 require 作为一个配置对象被定义时，我们希望一旦 require() 被定义就进行依赖加载。使用 deps 配置就像使用 ```require([])``` 一样，但是加载器一旦完成配置处理就会执行。**它不会阻碍任何其他 require() 的调用**，它仅仅是指明[配置对象](#configBlock)中那些模块进行异步加载。

**callback**：在 **deps** 指定的所有模块加载完成后执行的一个函数。在 require.js 加载前 require 作为一个配置对象被定义，同时我们希望在 **deps** 指定模块加载完成后再调用 require 函数加载某些模块时十分有用。

**enforceDefine**：设定为 true 后，如果一个被加载模块没有使用 defined() 进行定义或者 [shim exports](#configShim) 字符串对应的模块值没有被检测到，则会抛出一个错误。

**xhtml**：如果设置为true，则使用document.createElementNS()去创建script元素。

**urlArgs**：向 RequireJs 用于请求资源的 URLs 后添加查询字符串参数。主要用于在浏览器或服务器配置错误时进行破坏缓存。
```
urlArgs: "bust=" +  (new Date()).getTime()
```

自 RequireJs 2.0.0 起，urlArgs 可以是一个函数。如果是一个函数，它将会接收一个模块 ID 和 URL 作为参数，同时他应该返回一个字符串，这个字符串会被添加到 URL 末端。在添加 '?' 或 '&' 时要根据现有 URL 的状态决定如何添加。
```
requirejs.config({
    urlArgs: function(id, url) {
        var args = 'v=1';
        if (url.indexOf('view.html') !== -1) {
            args = 'v=2'
        }

        return (url.indexOf('?') === -1 ? '?' : '&') + args;
    }
});
```

**scriptType**：指定 RequireJs 将 script 标签插入 document 时所用的 type="" 值。默认为“text/javascript”。想要启用Firefox 的 JavaScript 1.8特性，可使用值 “text/javascript;version=1.8”。