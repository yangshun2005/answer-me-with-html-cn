---
title: TCP 三次握手
subtitle: 两台主机如何协商通信，以及如何结束通信
cols: 3
source: RFC 9293
---
TCP 通过三条消息建立连接，通过四条消息关闭连接。序列号让双方保持同步。

## A 三次握手 {span=2 meta="open"}
```sequence num
participants: Client, Server
note Server: LISTEN
Client -> Server: SYN, seq=x
note Client: SYN_SENT
Server -> Client: SYN+ACK, seq=y, ack=x+1
Client -> Server: ACK, ack=y+1
note Client, Server: ESTABLISHED
```

## B 为什么需要三条消息
1. 服务器得知客户端能够发送数据。
2. 客户端得知服务器能够发送和接收数据。
3. 服务器得知客户端能够接收数据。

```callout warn 两条不够
一个旧的重复 SYN 可能延迟到达。如果只有两条消息，服务器会为它打开一个无用的连接。
```

## C 状态变化 {span=2 meta="simplified"}
```flow LR
(CLOSED) -> LISTEN: passive open
LISTEN -> SYN_RCVD: get SYN / send SYN+ACK
(CLOSED) --> SYN_SENT: active open / send SYN
SYN_SENT -> *ESTABLISHED: get SYN+ACK / send ACK
SYN_RCVD -> *ESTABLISHED: get ACK
```

## D 标志位
| 标志位 | 作用 | 握手中的使用 |
|---|---|---|
| SYN | 开始连接 | ok 第 1、2 步 |
| ACK | 确认收到 | ok 第 2、3 步 |
| FIN | 停止发送 | no 未使用 |
| RST | 立即重置 | warn 出错时使用 |