---
title: "Service Worker Cookbook"
description: "本cookbook包含若干Service Worker相关技术的用例与相关API描述"
pubDate: "2023-07-21 19:06"
heroImage: "https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230801113904.png"
date created: 2023-07-21
date modified: 2023-07-21
tags:
    - writings
    - W3C
---

> The winner ain't the one with the fastest car it's the one who refuses to lose.  
> — <cite>Dale Earnhardt</cite>


:::tldr 
`Service Worker` 本质上充当 Web 应用程序、浏览器与网络（可用时）之间的代理服务器。这个 API 旨在创建有效的离线体验，它会拦截网络请求并根据网络是否可用来采取适当的动作、更新来自服务器的的资源。它还提供入口以推送通知和访问后台同步 API。  
:::

## TOC

## 基本概念

`Service Worker` 是一个注册在指定源和路径下的事件驱动 `worker`。它采用 JavaScript 文件的形式，控制关联的页面或者网站，拦截并修改访问和资源请求，细粒度地缓存资源。你可以完全控制应用在特定情形（最常见的情形是网络不可用）下的表现。

`Service Worker` 运行在 `worker` 上下文：因此它无法访问 `DOM`，相对于驱动应用的主 JavaScript 线程，它运行在其他线程中，所以不会造成阻塞。它被设计为完全异步；因此，同步 `XHR` 和 `Web Storage`不能在 `Service Worker` 中使用。

出于安全考量，`Service Worker` 只能由 `HTTPS` 承载，毕竟修改网络请求的能力暴露给中间人攻击会非常危险，如果允许访问这些强大的 API，此类攻击将会变得很严重。在 Firefox 浏览器的用户隐私模式，Service Worker 不可用。

:::tip
Service worker 大量使用 `Promise`，因为通常它们会等待响应后继续，并根据响应返回一个成功或者失败的操作。Promise 非常适合这种场景。 
:::

## 用法

通常遵循以下基本步骤来使用 service worker：

1. 获取 service worker 代码，然后使用 [`serviceWorkerContainer.register()`](https://developer.mozilla.org/zh-CN/docs/Web/API/ServiceWorkerContainer/register) 来注册。如果成功，service worker 将在 [`ServiceWorkerGlobalScope`](https://developer.mozilla.org/zh-CN/docs/Web/API/ServiceWorkerGlobalScope) 中执行；这本质上是一种特殊的上下文，在主脚本执行线程之外运行，没有访问 DOM 的权限。Service Worker 现在已为处理事件做好准备。
2. 安装完成。`install` 事件始终是发送给 service worker 的第一个事件（这可用于启动填充 IndexedDB 和缓存站点资源的过程）。在此步骤期间，应用程序正在为离线可用做准备。
3. 当 `install` 程序处理完成时，service worker 被视为已安装。此时，service worker 的先前版本可能处于激活的状态并控制着打开的页面。由于我们不希望同一 service worker 的两个不同版本同时运行，因此新版本尚未激活。
4. 一旦 service worker 的旧版本控制的页面都已关闭，就可以安全地停用旧版本，并且新安装的 service worker 将收到 `activate` 事件。`activate` 的主要用途是去清理 service worker 之前版本使用的资源。新的 service worker 可以调用 [`skipWaiting()`](https://developer.mozilla.org/zh-CN/docs/Web/API/ServiceWorkerGlobalScope/skipWaiting) 要求立即激活，而无需要求打开的页面关闭。然后，新的 service worker 将立即收到 `activate` 事件，并将接管任何打开的页面。
5. 激活后，service worker 将立即控制页面，但是只会控制那些在 `register()` 成功后打开的页面。换句话说，文档必须重新加载才能真正的受到控制，因为文档在有或者没有 service worker 的情况下开始存在，并在其生命周期内维护它。为了覆盖次默认行为并在页面打开的情况下，service worker 可以调用 [`clients.claim()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Clients/claim) 方法。
6. 每当获取新版本的 service worker 时，都会再次发生此循环，并在新版本的激活期间清理上一个版本的残留。

![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230721213529.png)

以下是可用的 service worker 事件：

- install
- activate
- message 
  受控页面可以使用 `ServiceWorker.postMessage()` 方法向 Service Worker 发送消息。 Service Worker 可以选择通过与受控页面相对应的 `Client.postMessage()` 发回响应。
- 功能性事件
    - fetch  
      当主应用线程发出网络请求时，会在 Service Worker 的全局范围内触发 `fetch` 事件。它能够拦截网络请求并发送自定义响应（例如，返回本地缓存）。
    - sync  
      当向 `SyncManager` 注册事件的页面（或工作线程）正在运行且网络连接可用时，将触发 `ServiceWorkerGlobalScope` 接口的 `sync` 事件。
    - push  
      当 Service Worker 收到推送消息时， `push` 事件将发送到 Service Worker 的全局范围（由 `ServiceWorkerGlobalScope` 接口表示）。

```js
const registerServiceWorker = async () => {
  if ("serviceWorker" in navigator) {
    try {
      const registration = await navigator.serviceWorker.register("/sw.js", {
        scope: "/",
      });
      if (registration.installing) {
        console.log("正在安装 Service worker");
      } else if (registration.waiting) {
        console.log("已安装 Service worker installed");
      } else if (registration.active) {
        console.log("激活 Service worker");
      }
    } catch (error) {
      console.error(`注册失败：${error}`);
    }
  }
};

// …

registerServiceWorker();

```

:::warning 
service worker 代码只是一个驻留在我们的 app 内的一个 JavaScript 文件（注意，这个文件的 URL 是相对于源（origin）的，而不是相对于引用它的那个 JS 文件）。  
:::

单个 service worker 可以控制很多页面。每个作用域（scope）里的页面加载完的时候，安装在页面的 service worker 就可以控制它。需要小心 service worker 脚本里的全局变量：每个页面不会有自己独有的 worker。

## 使用Service Worker离线缓存

`install` 事件会在注册成功完成之后触发。`install` 事件通常会这样用，将离线运行 app 产生的资源放置在浏览器离线缓存的空间。为了实现这个，可使用 Service Worker 的存储 API——`cache`——一个 service worker 上的全局对象，它可以存储网络响应发来的资源，并且根据它们的请求来生成 key。这个 API 和浏览器的标准的缓存工作原理很相似，但它特定于域。直到你清理它们之前，这些内容都会持久存在。

```js
const addResourcesToCache = async (resources) => {
    const cache = await caches.open("v1");
    await cache.addAll(resources);
};

self.addEventListener("install", (event) => {
    event.waitUntil(
        addResourcesToCache([
              "/",
              "/index.html",
              "/style.css",
              "/app.js",
              "/image-list.js",
              "/logo.jpg",
              "/gallery/1.jpg",
              "/gallery/2.jpg",
              "/gallery/3.jpg",
        ])
    );
});
```

1. 使用`install`事件监听器监听`service worker`（即self），接着在事件内调用`ExtendableEvent.waitUntil()`方法——确保Service Worker不会在`waitUntil`里面的代码执行完之前安装完成。
2. 在 `addResourcesToCache()` 内，使用了 [`caches.open()`](https://developer.mozilla.org/zh-CN/docs/Web/API/CacheStorage/open) 方法来创建了叫做 `v1` 的新缓存，这将会作为站点资源缓存的第 1 个版本。然后在创建的缓存中调用 `addAll()` 函数，它的参数采用一个 URL 数组，指向想要缓存的所有资源。其中，URL 是相对于 worker 的location)。
3. 如果 promise 被拒绝，安装就会失败，这个 worker 不会做任何事情。
4. 当安装成功完成之后，service worker 就会激活。

:::tip
 Web Storage API（`localStorage`）跟 service worker 的 cache 工作原理十分类似，但是它是同步的，所以不允许在 service worker 中使用。
:::

:::tip
也可以在 service worker 中使用 [IndexedDB](https://developer.mozilla.org/zh-CN/docs/Web/API/IndexedDB_API) 来做数据存储。
:::

通过上面一步，已将站点资源缓存了，但还需要告诉Service Worker让它用这些缓存内容做点啥。这个可通过`fetch`事件来处理：
1. 每次获取 service worker 控制的资源时，都会触发 `fetch` 事件，这些资源包括了指定的作用域内的文档，和这些文档内引用的其他任何资源（比如 `index.html` 发起了一个跨源的请求来嵌入一个图片，这个也会通过 service worker）。
2. 可以给 service worker 添加一个 `fetch` 的事件监听器，接着调用 event 上的 `respondWith()` 方法来劫持 HTTP 响应，然后可以用自己的方法来更新它们。
3. 在任何情况下，会首先响应缓存的 URL 和网络请求的 URL 相匹配的资源

```js
const putInCache = async (request, response) => {
    const cache = await caches.open("v1");
    await cache.put(request, response);
};

const cacheFirst = async ({ request, preloadResponsePromise, fallbackUrl }) => {
    // 首先，尝试从缓存中获取资源
    const responseFromCache = await caches.match(request);
    if (responseFromCache) {
        return responseFromCache;
    }

    // 然后尝试从网络中获取资源
    try {
        const responseFromNetwork = await fetch(request);
        // 响应可能会被使用
        // 需要将它的拷贝放入缓存
        // 然后再返回该响应
        putInCache(request, responseFromNetwork.clone());
        return responseFromNetwork;
    } catch (error) {
        const fallbackResponse = await caches.match(fallbackUrl);
        if (fallbackResponse) {
            return fallbackResponse;
        }
        // 当回落的响应也不可用时，便无能为力了，但需要返回 Response 对象
        return new Response("Network error happened", {
            status: 408,
            headers: { "Content-Type": "text/plain" },
        });
    }
};

self.addEventListener("fetch", (event) => {
    event.respondWith(
        cacheFirst({
            request: event.request,
            fallbackUrl: "/gallery/1.jpg",
        })
    );
});

```

`caches.match(event.request)` 允许对网络请求里的每个资源与缓存里可获取的等效资源进行匹配，查看缓存中是否有相应的资源。该匹配通过 URL 和各种标头进行，就像正常的 HTTP 请求一样。

![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230721215953.png)




## 注册失败的可能原因

- 没有在 HTTPS 下运行你的程序。
- service worker 文件的路径没有写对——需要相对于源（origin），而不是 app 的根目录。
- service worker不允许 app 指向不同源（origin）。
- service worker 只能在 service worker 作用域内捕获客户端发出的请求。
- service worker 最大的作用域是 worker 所在的位置（换句话说，如果脚本 `sw.js` 位于 `/js/sw.js` 中，默认情况下它只能控制 `/js/` 下的 URL）。可以使用 [`Service-Worker-Allowed`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Header/Service-Worker-Allowed "This is a link to an unwritten page") 标头指定 worker 的最大作用域列表。
- 在 Firefox 中，若用户处于[无痕浏览模式](https://support.mozilla.org/zh-CN/kb/private-browsing-use-firefox-without-history)、禁用了历史记录或者启用了在 Firefox 关闭时清除历史记录，Service Worker API 将被隐藏而无法使用。
- 在 Chrome 中，当启用“阻止所有 Cookie（不建议）”选项时，注册将会失败。


### Service Worker相关API

### Service Worker

`Service Worker API`的 **`ServiceWorker`** 接口提供了对 service worker 的引用。各个浏览上下文（例如页面、worker 等）可以与相同的 service worker 相关联，每个浏览上下文都可以通过唯一的 `ServiceWorker` 对象访问。

`ServiceWorker` 接口被分配了一系列生命周期事件——`install` 和 `activate`——以及功能型的事件，包括 `fetch`。一个 ServiceWorker 对象有一个与之关联的 `ServiceWorker.state`，指示着它的生命周期。

```js
/// 下面代码监听了任何`ServiceWorker.state`的改变，并在页面中显示其值

let serviceWorker;
if (registration.installing) {
  serviceWorker = registration.installing;
  document.querySelector('#kind').textContent = 'installing';
} else if (registration.waiting) {
  serviceWorker = registration.waiting;
  document.querySelector('#kind').textContent = 'waiting';
} else if (registration.active) {
  serviceWorker = registration.active;
  document.querySelector('#kind').textContent = 'active';
}

if (serviceWorker) {
  logState(serviceWorker.state);
  serviceWorker.addEventListener('statechange', function(e) {
  logState(e.target.state);
  });
}
```

### Fetch

Fetch 提供了对 [`Request`](https://developer.mozilla.org/zh-CN/docs/Web/API/Request) 和 [`Response`](https://developer.mozilla.org/zh-CN/docs/Web/API/Response)（以及其他与网络请求有关的）对象的通用定义。这将在未来更多需要它们的地方使用它们，无论是 service worker、Cache API，又或者是其他处理请求和响应的方式，甚至是任何一种需要在程序中生成响应的方式（即使用计算机程序或者个人编程指令）。

Fetch 的核心在于对 HTTP 接口的抽象，包括 [`Request`](https://developer.mozilla.org/zh-CN/docs/Web/API/Request)，[`Response`](https://developer.mozilla.org/zh-CN/docs/Web/API/Response)，[`Headers`](https://developer.mozilla.org/zh-CN/docs/Web/API/Headers)，`Body`，以及用于初始化异步请求的 [`global fetch`](https://developer.mozilla.org/zh-CN/docs/Web/API/fetch "global fetch")。得益于 JavaScript 实现的这些抽象好的 HTTP 模块，其他接口能够很方便的使用这些功能。

### Cache

**`Cache`** 接口为缓存的 [`Request`](http://fetch.spec.whatwg.org/#request) / `Response` 对象对提供存储机制，例如，作为[`ServiceWorker`](https://developer.mozilla.org/zh-CN/docs/Web/API/ServiceWorker) 生命周期的一部分。请注意，Cache 接口像 workers 一样，是暴露在 window 作用域下的。尽管它被定义在 service worker 的标准中，但是它不必一定要配合 service worker 使用。

一个域可以有多个命名 Cache 对象。你需要在你的脚本 (例如，在 [`ServiceWorker`](https://developer.mozilla.org/zh-CN/docs/Web/API/ServiceWorker) 中) 中处理缓存更新的方式。除非明确地更新缓存，否则缓存将不会被更新；除非删除，否则缓存数据不会过期。使用 [`CacheStorage.open(cacheName)`](https://developer.mozilla.org/zh-CN/docs/Web/API/CacheStorage/open "CacheStorage.open(cacheName)") 打开一个 Cache 对象，再使用 Cache 对象的方法去处理缓存。

需要定期地清理缓存条目，因为每个浏览器都硬性限制了一个域下缓存数据的大小。缓存配额使用估算值，可以使用 [`StorageEstimate`](https://developer.mozilla.org/zh-CN/docs/Web/API/StorageManager/estimate) API 获得。浏览器尽其所能去管理磁盘空间，但它有可能删除一个域下的缓存数据。浏览器要么自动删除特定域的全部缓存，要么全部保留。确保按名称安装版本缓存，并仅从可以安全操作的脚本版本中使用缓存。


### Push

**Push API** 给与了 Web 应用程序接收从服务器发出的推送消息的能力，无论 Web 应用程序是否在用户设备前台，甚至刚加载完成。这样，开发人员就可以向用户投放异步通知和更新，从而让用户能更及时地获取新内容。

对于一个应用来说，要想要接收到推送消息，需要有一个激活的 service worker。当 service worker 处于激活状态时，可以使用 `PushManager.subscribe()` 来订阅推送通知。

返回的 `PushSubscription`  包含了有关应用需要发送的推送消息的所有信息：端点及发送数据需要的加密密钥。

Service worker 会在必要的时候启动并接收传入的推送消息，将其传递给 `onpush`事件处理器。该方法允许应用程序在接收到推送消息时作出反应，例如显示一条通知（使用 `ServiceWorkerRegistration.showNotification()`）。

每一个订阅对 service worker 来说都是唯一的。同时订阅的端点也是一个唯一的功能性 URL：端点的信息是给应用发送信息的全部必要条件。所以端点地址需要保密，否则其他应用也可以向你的应用推送消息。

激活一个 service worker 来提供推送消息会导致资源消耗的增加，尤其是电池。不同的浏览器对此有不同的方案——目前为止还没有标准的机制。Firefox 允许对发送给应用的推送消息做数量限制（配额），但生成通知的推送消息不受此限制。该限制会在站点每一次被访问之后刷新。相比之下，Chrome 选择不作限制。

### Notifications

**Notifications API** 允许网页控制向最终用户显示系统通知 —这些都在顶级浏览上下文视口之外，因此即使用户已经切换标签页或移动到不同的应用程序，也可以显示。该 API 被设计成与不同平台上的现有通知系统兼容。

要显示一个系统通知，首先，用户需要授予当前源的权限以显示系统通知，这通常在应用或站点初始化时，使用`Notification.requestPermission()` 方法来完成。接下来，使用 `Notification()`)构造函数创建一个新通知。这个方法可以传入两个参数。这必须传递一个标题参数，并可以选择性地传递一个选项对象来指定选项，如文本方向，正文，显示图标，通知声音播放，等等。

```js
Notification.requestPermission( function(status) {
  console.log(status); // 仅当值为 "granted" 时显示通知
  const n = new Notification("title", {body: "notification body"}); // 显示通知
});

```

:::warning

此特性在 [Web Worker](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API) 中可用

:::

### BackgroundSync

`ServiceWorkerRegistration` 接口的 `sync` 属性返回对 `SyncManager` 接口的引用，该接口管理后台同步进程。

后台同步 API 使 Web 应用程序能够推迟任务，以便一旦用户拥有稳定的网络连接，它们就可以在 Service Worker 中运行。
例如，电子邮件客户端应用程序可以让用户随时撰写和发送消息，即使设备没有网络连接也是如此。应用程序前端仅注册同步请求，当网络再次存在并处理同步时，服务侧会收到警报。

以下示例，展示从浏览器上下文中注册一个标签为`sync-message`的后台同步任务
```js
async function syncMessagesLater() {
    const registration = await navigator.serviceWorker.ready;
    try {
        await registration.sync.register("sync-messages");
    } catch {
        console.log("Background Sync could not be registered!");
    }
}

```

检查指定标签的后台同步任务：
```js
navigator.serviceWorker.ready.then((registration) => {
    registration.sync.getTags().then((tags) => {
        if (tags.includes("sync-messages")) console.log("Messages sync already requested");
    });
});
```

在Service Work中响应后台同步事件：
```js
self.addEventListener("sync", (event) => {
    if (event.tag === "sync-messages") {
        event.waitUntil(sendOutboxMessages());
    }
});
```

[Introducing Background Sync - Chrome Developers](https://developer.chrome.com/blog/background-sync/)


## 示例

### Mock Service

:::info
更专业的库：[MSW – Seamless API mocking library for browser and Node | Mock Service Worker](https://mswjs.io/)
:::

```js
self.addEventListener('fetch', function (event) {
    console.log('Handling fetch event for', event.request.url);
    const requestUrl = new URL(event.request.url);

    if (requestUrl.pathname === '/urlshortener/v1/url' &&
        event.request.headers.has('X-Mock-Response')) {
        const responseBody = {
            kind: 'urlshortener#url',
            id: 'http://a.be/rqge13',
            longUrl: 'https://a-long-request.github.io/index.html'
        };

        const responseInit = {
            status: 200,
            statusText: 'OK',
            headers: {
                'Content-Type': 'application/json',
                'X-Mock-Response': 'yes'
            }
        };

        const mockResponse = new Response(JSON.stringify(responseBody), responseInit);

        console.log(' Responding with a mock response body:', responseBody);
        event.respondWith(mockResponse);
    }
});
```

### Local Download

通常，有必要在单页应用程序中包含下载功能 - 例如，绘图程序可能需要能够导出为 SVG 或生成客户端的位图格式。

使用Service Worker 拦截表单 POST 操作，从请求正文中提取数据。然后，可以将数据放入充当可下载附件的请求中，并将其作为文件反馈给客户端。该文件似乎已被下载，无需发送至服务器。
```js
self.addEventListener('fetch', function (event) {
    if (event.request.url.indexOf("download-file") !== -1) {
        event.respondWith(event.request.formData().then(function (formdata) {
            var filename = formdata.get("filename");
            var body = formdata.get("filebody");
            var response = new Response(body);
            response.headers.append('Content-Disposition', 'attachment; filename="' + filename + '"');
            return response;
        }));
    }
});
```
