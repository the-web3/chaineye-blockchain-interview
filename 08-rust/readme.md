# rust


## 1.详细说明 async-std，tokio，actix，native-tls，rustls 等的区别与联系

- async-std 和 tokio
async-std 和 tokio 都是 Rust 中用于异步编程的库。它们的主要区别在于，async-std 更注重提供类似于 Rust 标准库的 API 和使用习惯，而 tokio 则提供了更为灵活的 API 和自定义能力。在实际使用中，如果需要快速上手，可以选择 async-std；如果需要更高的灵活性和自定义能力，可以选择 tokio。

- tokio 和 actix
tokio 是一个异步编程框架，提供了用于异步 I/O 操作的基础设施，例如事件循环和异步任务的执行器。而 actix 是一个基于 tokio 的异步 Web 框架，提供了 Web 开发的相关功能和工具，例如 HTTP 服务器和路由等。

- native-tls 和 rustls
native-tls 和 rustls 都是 Rust 中用于 TLS 加密和解密的库。它们的主要区别在于，native-tls 使用操作系统的本地 TLS 库，例如 OpenSSL，而 rustls 是一个纯 Rust 实现的 TLS 库。在实际使用中，如果需要更高的性能和对操作系统 TLS 库的依赖度较低，可以选择 rustls；如果需要更广泛的兼容性和对操作系统 TLS 库的支持，可以选择 native-tls。

总的来说，async-std、tokio、actix、native-tls、rustls 这些库和框架都是 Rust 中非常常用的工具，用于异步编程和网络编程。它们之间的区别和联系需要根据具体的使用场景和需求来选择。


## 2. 简单说明可变变量以及不可变变量
