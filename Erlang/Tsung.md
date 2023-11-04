# tsung压测工具
- ssh集群互连免密互连
- 集群压测
- 监控服务器
- 常见问题

## 常见问题
- ssh <server_host> "erl"
   "Bash: erl command not found"

```shell
$ which erl
/usr/local/erlang/bin/erl
$ sudo ln -s /usr/local/erlang/bin/erl /usr/bin/erl
$ which tsung
/usr/local/erlang/bin/tsung
$ sudo ln -s /usr/local/erlang/bin/tsung /usr/bin/tsung
```