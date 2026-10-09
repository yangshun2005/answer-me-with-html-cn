---
title: 我们的缓存该用 Redis 还是 Memcached？
subtitle: 为一个 Web 应用提供会话和页面缓存
cols: 3
---
两者都是快速的内存存储。它们的区别在于重启后会发生什么，以及你能存储什么。

## A 功能对比 {span=2}
| 需求 | Redis | Memcached |
|---|---|---|
| 重启后保留数据 | ok RDB 快照、AOF 日志 | no 仅内存 |
| 数据类型 | ok 字符串、哈希、列表、集合 | warn 仅字符串 |
| 按 key 设置过期时间 | ok 支持 | ok 支持 |
| 复制 | ok 内置 | no 非内置 |
| 多 CPU 核心 | warn 单主线程 | ok 多线程 |
| 小 key 的内存占用 | warn 略高 | ok 更低 |

## B 结论
```callout ok 使用 Redis
会话必须在重启后存活，而购物车需要哈希。Memcached 两者都做不到。
```
只有在纯页面缓存、且随时可以重建的情况下，才选择 Memcached。

## C 我们的数据 {span=3}
```kv cols=4
峰值负载: 4,000 请求/秒
缓存大小: 约 2 GB
重启预算: 0 会话丢失
团队技能: 另外 2 个服务已使用 Redis
```