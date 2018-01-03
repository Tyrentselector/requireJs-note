# 运行机制

RequireJs 使用 ```head.appendChild()``` 向 head 中添加 script 标签加载每一个依赖。

RequireJs 等待所有依赖加载完成，然后计算模块定义函数的调用顺序，一旦依赖模块的定义函数完成调用，就会调用引用模块的定义函数。注意由于依赖模块的子依赖关系和网络加载顺序的缘故，他们的定义函数是无序执行的。

在服务端同步加载环境中使用 RequireJs 应该简单重写 require.load()。构建系统会帮我们完成这项工作，服务端 require.load 方法能够在 ```build/jslib/requirePatch.js``` 中找到。
