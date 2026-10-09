---
title: sequence
---
## 使用令牌登录
```sequence num
Browser -> API: POST /login
API -> Auth: check password
Auth --> API: token
API --> Browser: 200, set cookie
```