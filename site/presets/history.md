---
title: Kubernetes 是如何诞生的
subtitle: 从 Google 内部的集群管理器到运行容器的默认方式
cols: 3
---
## A 时间线 {span=3}
```timeline h
2003 | Google 的 Borg | 运行 Google 的大部分任务
2013 | Docker 发布 | 容器变得易于构建
*2014 | Kubernetes 发布 | 开源，用 Go 编写
2015 | v1.0 与 CNCF | Google 将项目交给基金会
2018 | 首个 CNCF 毕业项目 | 项目已稳定
2022 | v1.24 中移除 dockershim | 任何 CRI 运行时均可使用
```

## B Borg 带来了什么 {span=2}
| Borg 中的概念 | Kubernetes 中 |
|---|---|
| 按标签分组的任务 | 标签与选择器 |
| 同地协作任务的单元 | Pod |
| 每个任务组一个 IP 地址 | 每个 Pod 一个 IP 地址 |
| 中央状态存储 | etcd |

## C 为何得以普及
- 每家大型云厂商都将其作为托管服务提供。
- 无论是在笔记本电脑上还是在数据中心里，API 都是相同的。
- 扩展使用的 API 与核心相同。