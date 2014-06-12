router
======

router 模块由 [Page.js](http://visionmedia.github.io/page.js/) 修改而来，为单页面应用（[SPA](http://en.wikipedia.org/wiki/Single-page_application)）提供前端路由支持。

router 模块通过监听 hashchange 事件，对匹配一定模式的 hash 调用相应的处理函数进行处理。

``` javascript
var router = require('router');

router('/', index);
router('/user/:id', show);
router('/user/:id/edit', edit);
router('/user/:id/album', album);
router('/user/:id/album/sort', sort);
router('*', notfound);
router();

function index(ctx) {
    // ...
}

function show(ctx) {
    // ...
}
```

上述代码对一系列 URL hash 模式绑定了相应的处理函数，如：

``` javascript
router('/user/:id/edit', edit);
// 为形如 http://hostname/path/to/app#!/user/12/edit 的 URL 绑定处理函数 edit
```

在为 URL hash 模式绑定处理函数时，可以指定多个中间件以实现异步串行处理。

``` javascript
user = {
    load: function (ctx, next) {
        var id = ctx.params.id;
        net.get('/api/user?id=' + id, function (data) {
            ctx.state.data = data;
            next();
        });
    },
    show: funciton (ctx) {
        page.render(userTemplate, ctx.state.data);
    }
}

router('/user/:id', user.load, user.show);
```

在为 URL hash 绑定处理函数时，除最后一个函数外的中间件函数中都需要调用 next() 去触发后续处理函数的执行，多个处理函数间通过上下文对象 ctx 共享传递数据。

### router
router 函数支持下述几种用法：

* __router()__：等同于 `router.start()`
* __router(path, [state])__：等同于 `router.route(path, [state])`，state 为对象。
* __router(path, handler1, [handler2...])__：等同于 `router.bind(path, handler1, [handler2...])`，handler 为函数。

### context 上下文对象
context 上下文对象作为 URL hash 处理函数的第一个参数，包含了 URL hash 相关属性及状态信息，是多个处理函数间传递数据的桥梁。

以一段代码为例说明上下文对象各属性说明如下：

``` javascript
router('/user/:id/edit', edit);
```

假设当前 URL 为 `http://hostname/path/to/app#!/user/12/edit?a=1!!b=hello`

* __path__：`/user/12/edit?a=1!!b=hello`
* __target__：`#!/user/12/edit?a=1!!b=hello`（hashbang 可选）
* __pathname__：`/user/12/edit`
* __search__：`?a=1!!b=hello`
* __queries__：`{a: '1', b: 'hello'}` hash 中由 `!!` 分隔的类似 GET 参数的的参数对象
* __params__：`{id: '12'}` hash 中用 :word 形式捕获的参数
* __state__：`{}` 状态对象，用于多个函数间共享传递数据

__注意__ 由于片段标识符中不允许使用 `&` 符号，因此类 GET 参数需要使用 `!!` 分隔。

### router.start()
启动 router，开始监听 hashchange 事件。

### router.stop()
停止 router，取消监听 hashchange 事件。

### router.bind(pattern, handler1, [handler2, ...])
为满足 pattern 的 URL hash 绑定一个或多个处理函数。如：

``` javascript
router.bind('/user/:id/edit', user.load, user.edit)
```

在 pattern 中可以使用 `:variable` 的形式指定变量，绑定的处理函数在被调用时会被传入上下文对象 context。

在 pattern 中也可以使用 `*` 作为通配符匹配多个模式，如：

``` javascript
router.bind('/user/:id/*', user.load, user.process)
```

可以同时匹配 /user/:id/new、/user/:id/edit、/user/:id/delete 等。

### router.unbind(pattern, handler1, [handler2, ...])
解绑使用 `router.bind` 绑定到某 pattern 上的一个或多个处理函数。如：

``` javascript
router.unbind('/user/:id/edit', user.load, user.edit)
```

解绑时的 pattern 要与绑定时一致，要解绑的 handler 必须是所绑定 handler 的引用。

### router.route(path, [state], [dispatch])
改变当前 hash，跳转到 path 指定的 hash 地址，触发响应处理函数。参数如下：

* __path__：要跳转页面的 hash 地址，支持字符串`'/user/12/edit?a=1!!b=hello'`及数组`['/user', 12, 'edit', {a: 1, b: 'hello'}]`两种模式
* __state__：参数可选，要传递的状态对象，即上下文对象 ctx.state 属性
* __dispatch__：参数可选，是否触发响应处理函数

### router.replace(path, [state], [dispatch])
将当前 hash，替换为 path 指定的 hash 地址，触发相应处理函数，用法及参数与 `router.route` 相同，使用 `router.replace` 不会产生新的历史记录，即当使用 `router.replace` 替换 hash 后无法通过浏览器后退按钮返回被替换掉的状态。

### router.reset()
清空当前绑定的全部处理函数。
