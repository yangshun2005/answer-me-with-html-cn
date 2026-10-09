---
title: table
theme: blueprint
---
## A 选择哪个数据库？
| 需求 | Postgres | MongoDB | SQLite |
|---|---|---|---|
| 连接（Joins） | ok 支持 | warn 有限 | ok 支持 |
| 进程内运行 | no 不支持 | no 不支持 | ok 支持 |
| 水平扩展 | warn 需借助附加组件 | ok 内置支持 | no 不支持 |