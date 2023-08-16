---
title: "Rust入门之旅"
description: ""
pubDate: "2023-07-19 12:12"
heroImage: "https://plus.unsplash.com/premium_photo-1688114420848-1555448e8fa8?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&ixid=MnwxfDB8MXxyYW5kb218MHx8fHx8fHx8MTY4OTc1NTk2Ng&ixlib=rb-4.0.3&q=80&w=1200"
date created: "2023-07-19 16:41"
date modified: "2023-07-19"
tags:
    - notes
    - Programming
---

>草际烟光，水心云影，闲中观去，见乾坤最上文章 —《菜根谭》


## Rust如何实现零成本抽象

- 在一个`Scope` 下
    - 一个值只有一个所有者
    - 但可以有多个不可变引用
    - 以及唯一的可变引用（`mutual exclusive`）
    - 引用的生命周期不能超过值的生命周期
- 在多线程环境下
    - 类型安全（`Send` / `Sync`）保证并发安全

## Rust FFI

- C/C++/Swift: Cbindgen
- C++: autocxx
- Java: jni-rs、flapigen-rs、robusta
- Erlanh/Elixir：rustler
- Python: pyo3
- JavaScript: 
    - [neon-bindings/neon](https://github.com/neon-bindings/neon)
    - WebAssembly
    - [mozilla/uniffi-rs](https://github.com/mozilla/uniffi-rs) Mozilla的UniFFI库新增对JS绑定的生成支持（详情: https://hacks.mozilla.org/2023/08/autogenerating-rust-js-bindings-with-uniffi/）

## Rust 对网络协议的支持

![image.png](https://raw.githubusercontent.com/fw6/assets/main/toy_docs/20230719164743.png)


### 服务端应用的基本组成部分
- 数据序列化：serde、protobuf、flatbuffer、capnp、etc...
- 传输协议：tcp、http、websocket、quic、etc...
- 安全协议：TLS、noise protocol、secio、etc...
- 应用协议：application logic
- 数据在各个部分流转：shared memory、channel、etc...

