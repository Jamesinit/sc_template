# C interface

这个版本的模板程序会开启 DTP 的`interface` 功能，从而使得 DTP 可以与 C 语言的动态库进行连接，从而让他人可以通过实现 C 语言 API 的方法修改 DTP 的一些函数行为，而不需要修改源码。

## 编译与运行的注意事项

具有 `interface` 版本的 DTP 测试程序编译、运行起来相对来说复杂不少，下面是几个注意事项：

1. 首先在编译 Rust 库的时候需要开启`interface`参数：`cargo build --features="interface"
2. 然后在编译可执行文件的时候需要同时：1. 链接 `interface` 特性的 DTP 库。这个链接可以直接指定静态库文件 2. 利用给定的模板程序 `demo/.` 编译形成动态链接库，然后使用`-l`的方法链接编译。可以查看本目录下的 `Makefile` 文件进行进一步了解，也可以查看 https://github.com/simonkorl/rust_link_demo 来用一个简单的示例进行理解。
3. 最后在运行的时候，需要使用 `LD_LIBRARY_PATH=` 来指定动态链接库的位置。需要在程序运行时提供编译好的动态链接库程序才能正确运行。