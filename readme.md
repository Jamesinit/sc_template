# 测试用收发程序模板

使用模板程序和修改后的 DTP 和（或）其他的库文件即可完成测试用程序。

## 模板使用方法

你可以使用提供的模板程序生成对应的镜像以进行测试。

为了生成对应的镜像，你需要：

1. 在对应模板代码目录下，使用 `dockerfile_base` 生成一个基本的镜像。默认名字为 `sc_base`
    - 样例：`docker build . -f dockerfile_base -t sc_base`
2. 在修改后的 DTP 代码根目录下，使用 `dockerfile_full` 生成最终的测试用镜像。标签名可以自定。
    - 样例：`cd DTP && docker build . -f dockerfile_full -t test`
    - 建议同时添加 `.dockerignore` 文件来减少文件检测的问题。模板代码目录下的`.dockerignore`文件可以作为参考

或者可以：

1. 在 deps/DTP 目录下将 DTP 版本修改成需要的版本
2. 利用根目录下对应语言后缀的 dockerfile 来直接使用当前目录下的代码进行镜像的构建

## 收发程序一般性规范

### 服务端 server

负责数据的发送，一般可以支持同时给若干个不同的 client 进行发送

#### 输入

命令行接受三个参数：服务端的 IP 地址，监听的 Port，aitrans_block 配置数据文件。使用方法类似如下例子：

`./server 172.0.10.1 5555 aitrans_block.txt`

##### aitrans_block 格式

文件分为四列，分别含义如下：

| 距离上次发送的间隔时间(s) | 数据块 Deadline(ms) | 数据块大小 (B)| 数据块优先级 Priority|
|--|--|--|--|
| 0.01| 200 | 2000 | 1 |

```txt
0.015025005    200    1235    1
0.010311000000000002    200    288555    2
0.011021999999999997    200    1435    1
```

#### 输出

在 `./log/server_aitrans.log` 中输出一个文件，类似下面的示例

```txt
server start,  timestamp: 1638119474461087
connection closed, you can see result in client.log
1638119495831152: connection closed
recv,sent,lost,rtt(ns),cwnd
20760,22566,219,2130155,5840
total_bytes,total_time(us),throughput(B/s)
29550598,17965588,1738270
```

如果和[测试脚本](https://github.com/simonkorl/dtp_test_scripts)同步使用，需要在程序内**直接进行文件的书写**。测试脚本中会将 server 程序中所有命令行的输出均重定向到名为 `./log/server_err.log` 的文件中。

### 客户端 client

负责数据的接收并且打印最后的结果

#### 输入

命令行接受两个参数，分别代表 server 所对应的 IP 和 port。运行方式类似下方：

`./client 172.17.0.5 5555`

#### 输出

输出一个文件 `client.log` 其中的格式类似下方：

```txt
peer_addr = 172.17.0.5:5555
test begin!

BlockID  bct  BlockSize  Priority  Deadline
 5          1       1235          1        200
 9        165     288555          2        200
connection closed
recv,sent,lost,rtt(ms),cwnd,total_bytes,complete_bytes,good_bytes,total_time(us)
22347,20985,223,7.725466,14520,29267816,27969375,17018264,14974401

```

其中必须包括所有的块的信息。并且需要保证在一次正常运行时 `client.log` 的行数要大于 5，且不正常运行时 `client.log`  行数要不大于 5 。

如果和[测试脚本](https://github.com/simonkorl/dtp_test_scripts)同步使用，需要在程序内**直接进行文件的书写**。测试脚本中会将 client 程序中所有命令行的输出均重定向到名为 `./client_err.log` 的文件中。