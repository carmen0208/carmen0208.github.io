---
layout: post
title:  "服务器端渲染 Server Side Rendering"
date:   2020-05-30 08:26:03 +1200
tags: web programming
---
---
今天我们来讲讲server side rendering.
要理解清楚什么是Server Side Rendering，我们要先理解什么是Client Side Render

### 客户端渲染
Client Side Render是说我们的html的渲染工作放在了客户端，当页面加载的时候，我们的客户端用js来控制整个dom树的生成和事件处理，数据的加载。而最初加载的页面是一个包含对页面管理的js的引用，和dom树的根节点。
```html
<!DOCTYPE html>
<html>
  <head>
    <title>SPA</title>
  </head>
  <body>
	  <!-- root dom-->
    <div id="root"></div>
    <!-- spa javascript ->
	  <script type="text/javascript" src="/bundle.js"></script>
  </body>
</html>
```

以上边的代码为例，程序首先加载这个原始的html页面，加载完成之后，接下来的渲染工作，是由这个bundle js所引用的js代码执行过程中解析的。包括给整个dom树加子节点，数据提取。

客户端渲染CSR的好处是:

1. 页面的局部更新和加载
2. 第一次渲染之后的渲染都会很快。
3. 交互性更好

但是他带来了一些弊端：

1. 对SEO 不友好
2. 首屏加载慢

---
### Client Side Render

CSR is rendering an app in browser, generally using the DOM.
The initial HTML rendering by the server as a placeholder and the entire user interface and data rendering in the browser once all scripts loaded.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>SPA</title>
  </head>
  <body>
	  <!-- root dom-->
    <div id="root"></div>
    <!-- spa javascript ->
	  <script type="text/javascript" src="/bundle.js"></script>
  </body>
</html>
```

The steps like this:

1. User request the page by navigate the link
2. Server sending `html` response to browser
3. Browser downloads JS & Css
4. Browser execute the JS
5. Page now viewable and interactable

There’s some benefit by doing this:

1. Rich site interactiion
2. Fast rendering after the initial load
3. Partial real time update.
4. Can host in static CDN, cheaper to host & scale.

But with that, it also introduced some drawbacks like:

1. SEO and index issue.
2. inital bundle.js load duration
3. Performance issues on old mobile devices/slow networks.

---
### 服务器端渲染

那么我们怎么才能解决这两个问题呢，最好的方式是做服务器端渲染，也就是我们所说的SSR，做到服务器渲染我们需要做的事情是：

1. 创建服务器（server）来为页面做渲染，目前大多数框架选用Node + express。因为Node是js的跨平台运行时环境。服务器端的任务是：当接收请求的时候，服务器端根据请求返回渲染过后的页面。

    下例是要返回的html模板， 模板中${content}部分是要被服务器渲染和替换的部分

    ```html
    <!DOCTYPE html>
    <head>
            <!-- ************ -->
        </head>
        <body>
                <div id="root">${content}</div>
                <script src="/bundle.js"></script>
            </body>
    </html>
    ```

    后端做渲染必然会面临的问题是后端如何补足缺失的dom树结构和数据：而渲染本质对前后端来讲是一样的，都是字符串的拼接，将数据渲染进一些固定格式的html代码中形成最终的html展示在用户页面上。常见的手段，比如是直接生成DOM插入到html 中，或者是使用一些前端的模板引擎等。他们初次渲染的原理大多是将原html中的数据标记（例如{{text}}）替换。

    以react-dom为例，我们可以在服务器端调用reactdom的 renderToString方法，他就能将传入的component转化成 有dom树结构的string。

    ```javascript
    ReactDOMServer.renderToString(
        <StaticRouter>
        <App />
        </StaticRouter>
    )
    ```

    上边的代码生成的dom数长这样，如果涉及到对数据的提取。在渲染的时候，也需要懂api里边取到数据，并把这些数据结合到字符串中来。

    ```sh
    # output: 
    # <div><div><button>Posts</button><button>Todos</button></div><ul></ul></div>
    ```

2. 服务器会将渲染完成后的代码返回给浏览器，值得注意的是，这个被服务器端后端渲染的页面并不具有交互能力，也就是说，此时渲染出来的页面上边如果有跳转按钮，当点击跳转按钮的时候，是不会工作的，此时需要我们同时加载的客户端js bundle 代码。来执行这部分交互代码的渲染的补全，这个过程我们教工作hydrate

    Client.js
    ```javascript
    ReactDOM.hydrate(
    <Provider store={store}>
        <BrowserRouter>
        <div>{renderRoutes(Routes)}</div>
        </BrowserRouter>
    </Provider>,
    document.querySelector('#root')
    );
    ```

大家可以已经注意到，无论是客户端或则服务器端，我们用到的都是同部分代码做渲染，只是后端用renderToString 方法，前端用hydrate的方法而已，而这个区别正式为啥SSR又被叫做Isomorphic 同构或 Universal (普遍的，广泛的)的原因

---

### Server Side Render

To address those issues from CSR, that how SSR was introduced for. To do that, we have to making sure the following steps:

1. Having a server to render a page, take Next.js for example, it use node.js as the server and it task is to take request from client and returns a rendered page back to user.
	
    The following code is a html rendering templete. The place ${content} indicate where rendered content should replace.

    ```html
    <!DOCTYPE html>
    <head>
            <!-- ************ -->
        </head>
        <body>
                <div id="root">${content}</div>
                <script src="/bundle.js"></script>
            </body>
    </html>
    ```

    Take react-dom for example, it will provide a method `renderToString` for server. It will take a component as a argument and returns a string.

    Method call is like this:

    ```javascript
    ReactDOMServer.renderToString(
        <StaticRouter>
        <App />
        </StaticRouter>
    )
    ```

    And this is how  out put looks like :

    ```sh
    # output: 
    # <div><div><button>Posts</button><button>Todos</button></div><ul></ul></div>
    ```


2. Once the page have been rendered, it will returns to browser. But we need to notice that the page do not have any intractable, means that there’s no clickable, navigable event trigger. It need browser to load the js bundler to replenish all the interactive events calls Hydrate. Method looks like this:

    Client.js

    ```javascript
    ReactDOM.hydrate(
    <Provider store={store}>
        <BrowserRouter>
        <div>{renderRoutes(Routes)}</div>
        </BrowserRouter>
    </Provider>,
    document.querySelector('#root')
    );
    ```

Taking a close look, you may found that both server and client using the same kind of the component  fragment to rendering or hydrate the page. That the reason why SSR also calls Isomorphic or Universal Rendering as well.
