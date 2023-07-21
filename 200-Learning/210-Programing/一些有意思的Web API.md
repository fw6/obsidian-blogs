---
title: "有意思的Web API"
description: ""
pubDate: "2023-06-27 16:23"
heroImage: "https://images.unsplash.com/photo-1621839673705-6617adf9e890?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=640&q=80"
date created: 2023-06-27
date modified: 2023-06-27
tags:
    - notes
    - HTML
    - WebAPI
    - W3C
    - Programming
---

> The function of wisdom is to discriminate between good and evil.
> — <cite>Cicero</cite>

## TOC


## IME

- [IME | W3C](https://w3c.github.io/uievents/#ime)
- [text composition system | W3C](https://w3c.github.io/uievents/#text-composition-system)
- [Issue #180 · w3c/uievents · GitHub](https://github.com/w3c/uievents/issues/180)
- [Input method editor | MDN](https://developer.mozilla.org/en-US/docs/Glossary/Input_method_editor)
- [Understanding Composition Browser Events | Square Corner Blog](https://developer.squareup.com/blog/understanding-composition-browser-events/)

输入法编辑器是一种操作系统程序，用于将一种语言或字符集转换为另一种语言或字符集。例如输入*nihao*，键盘上没这种字符，但如果启用IME，开始打字时会出现下滑线表明IME处于活跃状态，在此过程中可以选择哪组字符与要输入的含义相匹配。按下Enter键将确认选择，将所选字符放到输入中，IME关闭。通过IME进行字符转换可输入来自任意国家/地区的母语。

## Edit Context

- [EditContext API](https://w3c.github.io/edit-context/)
![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230626145625.png)

![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230626145600.png)


## Background Fetch API

该API提供了一种新的方式在浏览器下载资源，如电影、音视频、软件。

当网友端要求客户下载大文件时，这通常需要用户保持与页面连接才能完成下载，如果失去连接、关闭选项卡、离开下载页面都会导致下载停止🤚。

[Background Synchronization API](https://developer.mozilla.org/en-US/docs/Web/API/Background_Synchronization_API) 使Web应用能延迟任务，一旦用户网络稳定即可在`Service Worker`中运行。但是它不能用于需要长时间运行的任务，例如下载大文件，后台同步要求Service保持活动状态直到完成，并为了节省电池寿命（节能模式）防止在后台产生不必要的任务，浏览器将在某个时刻终止该任务。

新的实验性技术`Background Fetch API`解决了该问题，它为Web开发人员提供了一种告诉浏览器在后台发送请求的方法。例如当用户点击下载按钮时下载视频时，浏览器以一种可见的方式执行`fetch`，并向用户显示进度和取消下载的方法，下载完成后，浏览器会打开`Service Worker`，此时应用程序可根据需要响应执行某些操作。
如果用户在离线状态下启动进程，`Background Fetch`也会启动，一旦网络连接，它就会开始，如果用户再次离线，会暂停知道用户再次上线。

### Example

```js
// client.js

function startBgFetch() {
    if (!('BackgroundFetchManager' in self)) {
        fallbackFetch(item);
        return;
    }
    const reg = await navigator.serviceWorker.ready;
    const bgFetch = await reg.backgroundFetch.fetch(
        id, 
        [item.src], 
        {
            title: item.title,
            icons: [
                { 
                    sizes: '300x300', 
                    src: item.image, 
                    type: 'image/jpeg' 
                }
            ],
            downloadTotal: item.size
        }
    );
    monitorBgFetch(bgFetch);
}

async function monitorBgFetch(bgFetch) {
    function doUpdate() {
        const update = {};
        if (bgFetch.result === '') {
            update.state = 'fetching';
            update.progress = bgFetch.downloaded / bgFetch.downloadTotal;
        } else if (bgFetch.result === 'success') {
            update.state = 'fetching';
            update.progress = 1;
        } else if (bgFetch.failureReason === 'aborted') { // Failure
            update.state = 'not-stored';
        } else { // other failure
            update.state = 'failed';
        }
    
        updateItem(bgFetch.id, update);
    };
      
      doUpdate();
    
      bgFetch.addEventListener('progress', doUpdate);
      const channel = new BroadcastChannel(bgFetch.id);
      
    channel.onmessage = (event) => {
        if (!event.data.stored) return;
        bgFetch.removeEventListener('progress', doUpdate);
        channel.close();
        updateItem(bgFetch.id, { state: 'stored' });
    };
}

// sw.js

addEventListener('fetch', (event) => {
    event.respondWith(async function() {
        const cachedResponse = await caches.match(event.request);
        return cachedResponse || fetch(event.request);
    }());
});

addEventListener('backgroundfetchsuccess', event => {
    const bgFetch = event.registration;
    event.waitUntil(async function () {
    new BroadcastChannel(bgFetch.id).postMessage({ stored: true });
    }());
});

addEventListener('backgroundfetchfail', event => {
    console.log('Background fetch failed', event);
});

addEventListener('backgroundfetchclick', event => {
    clients.openWindow('/');
});
```


