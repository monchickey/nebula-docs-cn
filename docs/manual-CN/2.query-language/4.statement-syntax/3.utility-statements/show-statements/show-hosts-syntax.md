# SHOW HOSTS 语法

```ngql
SHOW HOSTS
```

`SHOW HOSTS` 列出元服务器注册的所有存储主机。`SHOW HOSTS` 输出以下列：IP 地址、端口号、状态（online/offline）、leader 数量、leader 分布、partition 分布。

```ngql
nebula> SHOW HOSTS;
==============================================================================================
| Ip         | Port  | Status | Leader count | Leader distribution  | Partition distribution |
==============================================================================================
| 172.28.2.1 | 44500 | online | 9            | basketballplayer: 9  | basketballplayer: 10   |
----------------------------------------------------------------------------------------------
| 172.28.2.2 | 44500 | online | 0            |                      | basketballplayer: 10   |
----------------------------------------------------------------------------------------------
| 172.28.2.3 | 44500 | online | 1            | basketballplayer: 1  | basketballplayer: 10   |
----------------------------------------------------------------------------------------------
| Total      |       |        | 10           | basketballplayer: 10 | basketballplayer: 30   |
----------------------------------------------------------------------------------------------
```