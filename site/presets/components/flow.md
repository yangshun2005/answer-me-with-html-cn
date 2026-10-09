---
title: flow
---
## 请求路径
```flow LR
(User) -> Gateway: HTTPS
Gateway -> Auth & *Service
Service -> [(Database)]
group Backend: Auth, Service
```