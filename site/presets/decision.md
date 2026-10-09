---
title: 规划缓存变更
subtitle: 将页面缓存从进程内存迁移到 Redis
cols: 3
---
## A 现状 {meta="src/cache.ts"}
```ts title="cache.ts · 现状"
const pages = new Map<string, string>()

export function getPage(key: string) {
  return pages.get(key)
}

export function setPage(key: string, html: string) {
  pages.set(key, html)
}
```
每台服务器各自保留一份副本。一次部署就会将其清空。

## B 提议方案 {span=2 meta="sketch"}
```ts title="cache.ts · 草图"
import { createClient } from 'redis'

const redis = createClient({ url: process.env.REDIS_URL })
const TTL = 600 // seconds

export async function getPage(key: string) {
  return redis.get(`page:${key}`)
}

export async function setPage(key: string, html: string) {
  await redis.set(`page:${key}`, html, { EX: TTL })
}
```

## C 步骤 {span=2}
1. 添加 Redis 客户端，并在启动时读取 `REDIS_URL`。
2. 将这两个函数改为异步，并更新 4 个调用方。
3. 在功能开关后发布，并对比一天的命中率。
4. 移除内存中的 Map。

## D 待定决策
```ask
页面在缓存中保留多长时间？
* 10 分钟 | 与当前的部署节奏一致
- 1 小时 | 未命中更少，但页面更旧
- 直到下次部署 | 需要额外的清除步骤
```

```ask multi
除了页面，我们还缓存什么？
* 会话 | 在部署后依然保留
- API 响应 | 占用更多内存
```