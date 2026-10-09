---
title: TCP 三次握手
subtitle: 为什么是三步，而不是两步
---
> 在 TCP 发送数据之前，每一方都必须知道对方能够发送和接收。

## 双方等待
```sequence
Client -> Server: SYN
Server -> Client: SYN-ACK
Client -> Server: ACK
note Client, Server: ESTABLISHED
```
> 客户端发送 SYN 请求建立连接。
> [服务器]以 SYN-ACK 应答。
> 客户端回复 ACK。
> 现在双方都处于 ESTABLISHED 状态。

## 每一步证明了什么
```flow LR
Client -> Server: 1st: client can send
Server -> Client: 2nd: server can send and receive
Client -> *Both confirmed: 3rd: client can receive
```
> 第一步之后，服务器知道客户端能够发送。
> 第二步之后，客户端知道服务器能够双向通信。
> 只有第三步才能证明[客户端能够接收]。

## 如果只有两步会怎样
| 情况 | 两步 | 三步 |
|---|---|---|
| 一个旧的 SYN 迟到到达 | no 服务器会打开一个死连接 | ok 客户端不发送 ACK |
| 浪费资源 | no 空闲连接 | ok 无 |
| 双向检查 | warn 仅一半 | ok 完整 |
> 假设一个旧的 SYN 迟到到达。
> 如果是两步，服务器会打开一个没人使用的连接。
> 如果是三步，客户端不发送 ACK，因此不会打开连接。