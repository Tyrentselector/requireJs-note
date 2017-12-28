# 定义一个模块

模块与传统的脚本文件不同因为它定义了一个独立的作用域对象以避免污染全局命名空间。它可以明确的列出它的依赖，并且在获得那些依赖的句柄的同时不会涉及到全局对象，而是将依赖作为定义模块函数的参数进行接收。在 RequireJs 中模块扩展了[Module Pattern](http://www.adequatelygood.com/2010/3/JavaScript-Module-Pattern-In-Depth)，无需全局对象便可引用其他模块。

RequireJs 使得模块能够迅速加载，是指是无序加载，但是会评估依赖顺序的正确性，并且不会去创建全局变量，这使得在[同一页面中加载多个相同模块]()成为可能。

在每个模块文件中应该只出现一个模块定义。通过[优化工具](http://requirejs.org/docs/optimization.html)可以分组打包优化模块。

## 简单键值对模块定义

如果模块没有任何依赖，同时它仅是一些键值对，这时只需要传入这个简单对象进行定义：
```
// my/shirt.js:
define({
    color: "black",
    size: "unisize"
});
```

## 定义函数

如果模块没有任何依赖，但是需要一个函数作一些前置工作，这时传入一个函数进行定义：
```
//my/shirt.js now does setup work
//before returning its module definition.
define(function () {
    //Do setup work here

    return {
        color: "black",
        size: "unisize"
    }
});
```

## 定义带有依赖的函数

如果模块带有依赖，第一个参数应该是依赖名数组，第二参数应该为定义行数，一旦所有依赖加载完成，该函数将会被调用用于定义模块。该函数应该包含一个对象返回值。依赖会作为参数传递给定义函数，参数顺序同依赖顺序相同：
```
// my/shirt.js 现在拥有一些依赖, cart 和 inventory
// 模块与 shirt.js 在同一目录
define(["./cart", "./inventory"], function(cart, inventory) {
        // 返回一个对象定义 "my/shirt" 模块
        return {
            color: "blue",
            size: "large",
            addToCart: function() {
                inventory.decrement(this);
                cart.add(this);
            }
        }
    }
);
```

在该例子中创建了一个 **my/shirt** 模块。它依赖 **my/cart** 和 **my/inventory** 模块。目录结构如下：
```
my/cart.js
my/inventory.js
my/shirt.js
```

该函数调用以上两个特殊参数，**cart** 和 **inventory**。它们是 **./cart** 和 **./inventory** 模块名代表的模块。

直到 **./cart** 和 **./inventory** 两个模块加载完成该函数才会被调用。同时该函数会接收 **cart** 和 **inventory** 参数。 

由于不定义全局模块，所以同一模块的多个版本可以同时存在同一页面中。同时函数参数的顺序同依赖顺序相对应。

通过调用 **my/shirt** 模块的定义函数并返回一个对象的方式，**my/shirt** 不会纯在于全局对象中。

## 将一个模块定义为函数

模块不一定非要返回一个对象。返回任何一个合法的函数返回值都是可以的。下面是一个返回值为函数的模块：
```
// 在 foo/title.js 中定义了一个模块。它使用
// 上面的 my/cart 和 my/inventory 模块,
// 但是由于 foo/title.js 同 "my" 模块在不同的目录
// RequireJs 会尝试使用 "my" 定位这两个模块
// "my" 可以被映射至任何目录，但在默认情况下, 假设
// 它与 "foo" 在同一目录。
define(["my/cart", "my/inventory"],
    function(cart, inventory) {
        return function(title) {
            return title ? (window.title = title) :
                   inventory.storeName + ' ' + cart.name;
        }
    }
);
```

## 使用简化的 CommonJS 规范定义模块

如果我们想要使用一些使用传统 CommonJS 格式定义魔模块，修改依赖数组可不是一项轻松的工作，同时你可能更喜欢将依赖作为本地变量使用：
```
define(function(require, exports, module) {
        var a = require('a'),
            b = require('b');

        //Return the module value
        return function () {};
    }
);
```

## 定义一个含有模块名称的模块

我们可能偶尔会遇到某些 **define()** 调用，它包含一个模块名作为第一个参数：
```
// 定义 "foo/title" 模块:
    define("foo/title",
        ["my/cart", "my/inventory"],
        function(cart, inventory) {
            //Define foo/title object in here.
       }
    );
```

我们可以指定模块的名称，但是这样会使得模块灵活性降低。如果我们将文件从一个目录移动至另一个目录我们就需要改变相应名称。最好避免为一个模块进行命名，并且允许优化工具生成模块名。优化工具需要添加模块名以便多个模块可以在打包在同一文件中，以便提升在浏览器中的加载速度。

## 关于模块的其他说明

**每个文件只含有一个模块**：在一个文件中应该只定义一个模块。我们应该只使用优化工具将多个模块打包成一个优化文件。

**define() 中的相对模块名**：require("./relative/name") 可以在 define() 的回调函数中进行调用，确保 **require** 作为一个依赖引入，以便正确转变相对名称：
```
define(["require", "./relative/name"], function(require) {
    var mod = require("./relative/name");
});
```
或者直接使用 CommonJS 语法定义：
```
define(function(require) {
    var mod = require("./relative/name");
});
```
这种形式将会使用 **Function.prototype.toString()** 来定位 **require()** 的调用，并且将请求参数同 **require** 添加到依赖数组中，所以代码可以正确执行。

当我们在目录中创建模块时相对路径是非常实用的，因为我们可以将这个目录分享给其他人或者其他项目，同时我们可以在同目录中使用兄弟模块不用关心目录路径。

**相对模块名是相对于其他模块名而不是路径**：加载器通过模块名储存模块而不是通过模块路径存储模块。因此引用的相对名是相对于引用模块的模块名进行解析，如果需要加载再将模块名或模块 ID 转换成路径。例如对于 "compute" 包来说它含有 'main' 和 'extras' 模块：
```
* lib/
    * compute/
        * main.js
        * extras.js
```
main.js 代码如下：
```
define(["./extras"], function(extras) {
    //Uses extras in here.
});
```
如果使用以下 **paths** 配置：
```
require.config({
    baseUrl: 'lib',
    paths: {
      'compute': 'compute/main'
    }
});
```
并且当 **require(['compute'])** 函数执行结束，**lib/compute/main.js ** 将会获得 'compute' 模块名。当 **main.js** 请求 './extras' 时，会相对 'compute' 进行解析，所以 'compute/./extras' 会标准化为 'extras'。因为这里没有为 'extras' 配置 **paths** 所以会将 'lib/extras.js' 作为路径，而这个路径却是错误的。

对于这种情况，最好使用 [packages 配置]()，因为它允许设置 **main** 模块作为 'compute'，但是在内部加载器会使用模块 ID ('compute/main') 储存模块，所以对于 './extras' 的相对引用是可用的。

另一种不用配置 **paths** 或 **packages** 的方式是在 'lib' 下创建 'compute.js' 文件内容是 ```define(['./compute/main'], function(m) { return m; });```。

或不配置 **paths** 或 **packages** 在顶级调用 *require* 时使用 **require(['compute/main'])**。

相对于模块生成 URLs：我们可能会想相对某个模块生成一个 URL。将 "require" 作为依赖引入然后使用 **require.toUrl()** 来生成 URL：
```
define(["require"], function(require) {
    var cssUrl = require.toUrl("./style.css");
});
```

**控制台调试**：如果想使用在控制台中通过 **require(["module/name"], function(){})** 的方式加载过的模块，我们可以通过 **require()** 模块名的方式获取它：
```
require("module/name").callSomeFunction()
```

注意只有当 "module/name" 通过 **require** 的异步版本 "require(["module/name"])" 预先加载过以上方式才会生效。相对路径 './module/name' 只在 **define** 内部生效。

## 循环依赖

如果我们定义一个循环依赖（a 依赖 b，b 又依赖于 a），在这种情况下当 b 模块的回调执行后，这时 a 的值为 undefined。在模块定义完成后使用 ** require()** b 可以获取 a（注意确保指定 require 作为依赖项，以确保加载 a 时上下文正确）：
```
// b.js:
define(["require", "a"],
    function(require, a) {
        // 如果 "a" 同样依赖于 "b"，"a" 为 "null"
        return function(title) {
            return require("a").doSomething();
        }
    }
);
```

通常我们并不需要使用 **require()** 来获取模块，而是依赖作为参数传递给回调函数中的模块。循环依赖是一种很罕见的情况，通常遇到这种情况后我们要重新思考设计是否合理。然而，有些时候我们确实需要用到这种方法。

如果你更喜欢 CommonJS 模块，你可以使用 **export** 为模块创建一个空对象，这个空对象可以立即被其它引用模块使用。在俩个循环依赖两边这样做，我们就能获取其它模块了。这种方式只适用于每个模块导出的值为一个对象（function 不起作用）的情况：
```
//Inside b.js:
define(function(require, exports, module) {
    // 如果 a 使用 exports, 那么我们就拥有了一个
    // 引用对象。 然而, 我们现在还不能使用 a 的
    // 任何属性直到 "b" 模块返回一个值后。
    var a = require("a");

    exports.foo = function () {
        return a.bar();
    };
});
```

如果你使用的是依赖数组的方式，需要 **exports** 作为依赖：
```
//Inside b.js:
define(['a', 'exports'], function(a, exports) {
    // 如果 a 使用 exports, 那么我们就拥有了一个
    // 引用对象。 然而, 我们现在还不能使用 a 的
    // 任何属性直到 "b" 模块返回一个值后。

    exports.foo = function () {
        return a.bar();
    };
});
```

## 详细说明 JSONP 服务依赖

[JSONP](https://en.wikipedia.org/wiki/JSON#JSONP) 是在 JavaScript 中调用某种服务的方式。它可以进行跨域，也是一种通过 script 标签 HTTP GET 请求调用服务的方式。

在 RequireJs 中使用 JSONP 服务，将 **define** 作为回调参数值。这意味着我们可以得到 JSONP URL 的值，就好像它是一个定义模块一样。

下面是一个调用 JSONP API 的例子。在这个例子中 JSONP 回调函数参数为 **callback**，**callback=define** 告诉服务端 API 将 JSONP 请求包裹在 **define()** 内。
```
require(["http://example.com/api/data.json?callback=define"],
    function (data) {
        // data 对象就是 API 对 JSONP 请求的响应
        console.log(data);
    }
);
```

JSONP 服务的使用应该被限定在初始应用程序设置的 JSONP 服务上。如果 JSONP 服务超时，就意味着我们通过 **define()** 定义的其他模块可能不会被执行，所以错误处理不是很健壮。

**只支持 JSONP 返回值为 JSON 对象的情况**。当 JSONP 的返回值为 array、number 等其他值是都不会生效。

不要将该功能用于长轮询 JSONP 连接中——处理实时流媒体的 API。这种 API 在接收到每个请求后应该作脚本清理，同时 RequireJs 只能一次取回一个 JSONP URL——以后在 require() 或 define() 中使用相同的 URL 作为依赖的，将会得到缓存值。

在加载 JSONP 服务过程中遇到的错误通常表面上是服务超时，因为脚本加载没有给出更多网络问题的细节。为了侦测错误，我们可以通过复写 require.js.onError() 来获取错误。关于这部分信息我们可以参考[错误处理]()。

## 解除一个模块的定义

**requirejs.undef()** 为一个全局对象，它可以解除一个模块定义。它会重置加载器内部状态是得加载器忘记之前定义过的模块。

当它执行后它并不会从其它已经定义过的模块中移除解除定义的模块，同样也不会从将它作为依赖的模块中移除它。
