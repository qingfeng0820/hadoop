# Depends
```
nettyVersion = '4.1.5.RELEASE'
compile "io.netty:netty-all:${nettyVersion}"

只需要最基本的NIO传输功能,只需要引入依赖
compile "io.netty:netty-transport:${nettyVersion}"
此依赖会通过传递依赖的形式自动引入
netty-buffer,netty-common,netty-resolver,netty-transport
```
