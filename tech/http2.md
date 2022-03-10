



## 特性

### 多路复用

同个域名下的请求都是同个一个 TCP 连接完成的：

1. 单个连接可以承载任意数量的双向数据流
2. 数据流以消息的形式发送，消息又是以一个或多个帧组成，多个帧可以乱序发送
3. 每个请求可以带一个 31 bit 的优先值

### 服务器推送

服务端可以主动推送，客户端也有权利选择是否接收。如果服务端推送的资源已经被浏览器缓存过，浏览器可以通过发送RST_STREAM帧来拒收。主动推送也遵守同源策略，服务器不会随便推送第三方资源给客户端



## 技术细节

### 10 种类型的帧

- `HEADERS`: 报头帧 (type=0x1)，用来打开一个流或者携带一个首部块片段
- `DATA`: 数据帧 (type=0x0)，装填主体信息，可以用一个或多个 DATA 帧来返回一个请求的响应主体
- `PRIORITY`: 优先级帧 (type=0x2)，指定发送者建议的流优先级，可以在任何流状态下发送 PRIORITY 帧，包括空闲 (idle) 和关闭 (closed) 的流
- `RST_STREAM`: 流终止帧 (type=0x3)，用来请求取消一个流，或者表示发生了一个错误，payload 带有一个 32 位无符号整数的错误码 ([Error Codes](https://httpwg.org/specs/rfc7540.html#ErrorCodes))，不能在处于空闲 (idle) 状态的流上发送 RST_STREAM 帧
- `SETTINGS`: 设置帧 (type=0x4)，设置此 `连接` 的参数，作用于整个连接
- `PUSH_PROMISE`: 推送帧 (type=0x5)，服务端推送，客户端可以返回一个 RST_STREAM 帧来选择拒绝推送的流
- `PING`: PING 帧 (type=0x6)，判断一个空闲的连接是否仍然可用，也可以测量最小往返时间 (RTT)
- `GOAWAY`: GOWAY 帧 (type=0x7)，用于发起关闭连接的请求，或者警示严重错误。GOAWAY 会停止接收新流，并且关闭连接前会处理完先前建立的流
- `WINDOW_UPDATE`: 窗口更新帧 (type=0x8)，用于执行流量控制功能，可以作用在单独某个流上 (指定具体 Stream Identifier) 也可以作用整个连接 (Stream Identifier 为 0x0)，只有 DATA 帧受流量控制影响。初始化流量窗口后，发送多少负载，流量窗口就减少多少，如果流量窗口不足就无法发送，WINDOW_UPDATE 帧可以增加流量窗口大小



## k8s spdy

Dialer.Dial:

negotiate req:  {POST https://cls-rbu7mmt3.ccs.tencent-cloud.com/api/v1/namespaces/nh30sspg/pods/e-micro-agile-647dff77df-8npxq/portforward HTTP/1.1 1 1 map[X-Stream-Protocol-Version:[portforward.k8s.io]] <nil> <nil> 0 [] false cls-rbu7mmt3.ccs.tencent-cloud.com map[] map[] <nil> map[]   <nil> <nil> <nil> 0xc0000405d8}]"}

negotiate resp: %v {101 Switching Protocols 101 HTTP/1.1 1 1 map[Connection:[Upgrade] Date:[Mon, 18 Jan 2021 08:08:02 GMT] Upgrade:[SPDY/3.1] X-Stream-Protocol-Version:[portforward.k8s.io]] {} 0 [] false false map[] <nil> <nil>}]"}

在 negotiate 时，先发起一个 http1.1 的，协商一个 spdy 连接，在得到的 http1.1 resp 中，服务端会告诉客户端使用的协议（如 SPDY/3.1），通过该 resp，创建一个 SPDY client connection，类型是 httpstream.Connection。

创建一个 tcp server 连接，监听本地端口。

当本地端口收到请求时，先创建一个 error Stream，和远端建立 连接，

再建立一个 Data Stream，将远端的数据流拷到本地：

```go
// Copy from the remote side to the local port.
		if _, err := io.Copy(conn, dataStream); err != nil && !strings.Contains(err.Error(), "use of closed network connection") {
			runtime.HandleError(fmt.Errorf("error copying from remote stream to local connection: %v", err))
		}
```

将本地数据流拷到远端：

```go
// Copy from the local port to the remote side.
		if _, err := io.Copy(dataStream, conn); err != nil && !strings.Contains(err.Error(), "use of closed network connection") {
			runtime.HandleError(fmt.Errorf("error copying from local connection to remote stream: %v", err))
			// break out of the select below without waiting for the other copy to finish
			close(localError)
		}
```







关闭网络不会报“Lost connect to pod”，但切换网络会？