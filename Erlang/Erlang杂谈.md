title: Erlang杂货店
speaker: 飓风
js:
    - https://echarts.cdn.apache.org/zh/asset/theme/infographic.js
plugins:
    - echarts: {theme: infographic}
    - mermaid: {theme: forest}
    - katex

<slide>

:::{.aligncenter}
# Erlang杂货店{.text-landing.text-shadow}
By 飓风 {.text-intro.animated.fadeInUp.delay-500}

[:fa-github: Github](https://github.com/yangcancai){.button.ghost}
:::
:::footer
[:fa-twitter: 飓风](){.alignright}
:::

:::header
!![](https://www.erlang.org/assets/img/erlang-logo.svg .size-20)
:::

<slide class="aligncenter">


# 1.rebar3 如何依赖本地代码
- `cd <NameApp> && mkdir _checkouts`
- `ln -s <SourceProj>/depsA _checkouts/depsA`
-  增加 dpesA到 `rebar.config ` 依赖中
- rebar.config中deps已经更新到_build中的依赖会覆盖_checkouts

<slide class="aligncenter">
 
# 2. rust+erlang

- Rustler\: 一个运行时安全桥接Erlang和Rust的Rust库
- 数据结构转换
- 如何从Rustler线程返回数据给Erlang的某个进程
- 是否可以实现rust的actor和Erlang进程之间通信

<slide>

# 3.mnesia分片(mnesia fragment)

- mnesia多节点创建流程
- mnesia添加节点
- 分片表添加节点和分片
- 删除节点
- 删除分片

<slide>

## 3.1 mnesia多节点创建流程

```erlang
setup(L) ->
  %% first
  mnesia:create_schema(L),
  %% second
  [ rpc:call(Node, mnesia, start,[]) || Node <- L],
  %% create table
  {atomic, ok} = mnesia:create_table(user,
    [{ram_copies, L},
      {attributes, record_info(fields,
        user)}]),
  ok.
%% 创建分片表
  create_frag() ->
  _L = ['im_frag1@127.0.0.1', 'im_frag2@127.0.0.1'],
  {atomic, ok} = mnesia:create_table(log,
    [
      {frag_properties, [{n_fragments, 16}, {n_disc_copies, 2}]},
      {attributes, record_info(fields,
        log)}]),
  ok.

```

<slide>

## 3.2 mnesia添加节点

```erlang
add_new_node(N) ->
  mnesia:change_config(extra_db_nodes, [N]),
  mnesia:change_table_copy_type(schema, N, disc_copies).
```

<slide>

## 3.3 分片表添加节点和分片

-  分片添加节点前需要新的节点加入mnesia

```erlang
mnesia:change_table_flag(log, {add_node, 'im_frag3@127.0.0.1'}).
mnesia:change_table_flag(log, {add_frag, ['im_frag3@127.0.0.1', 'im_frag4@127.0.0.1']}).
```

<slide>

# 4. 使用Erlang实现一个虚拟机(VM in VM)
- 使用LC3指令集
- 在虚拟机运行一个汇编写的2048游戏

<slide> 

# rabbitmq启动参数

```shell
 /usr/lib/erlang/erts-12.0.3/bin/beam.smp -W w -MBas ageffcbf -MHas ageffcbf -MBlmbcs 512 -MHlmbcs 512 -MMmcs 30 -P 1048576 -t 5000000 -stbt db -zdbbl 128000 -sbwt none -sbwtdcpu none -sbwtdio none -B i -- -root /usr/lib/erlang -progname erl -- -home /var/lib/rabbitmq -- -pa  -noshell -noinput -s rabbit boot -boot start_sasl -lager crash_log false -lager handlers
```

<slide>

# asdf安装erlang配置参数

```shell
KERL_RELEASE_TARGET="lcnt valgrind gprof"
KERL_CONFIGURE_OPTIONS="--without-javac --enable-hipe --enable-kernel-poll --without-wx --without-odbc --enable-threads --enable-sctp --enable-smp-support --with-ssl=/usr/local/opt/openssl@1.1"
```

## Rocky 8.8 install
```shell
$ vim /root/.asdf/plugins/erlang/kerl-home/builds/asdf_24.2.1/otp_src_24.2.1/erts/config.log
$ ./configure --without-javac --enable-hipe --enable-kernel-poll --without-wx --without-odbc --enable-threads --enable-smp-support --disable-option-checking --cache-file=/dev/null --srcdir=/root/.asdf/plugins/erlang/kerl-home/builds/asdf_24.2.1/otp_src_24.2.1/erts
```

## Macos install
```shell
$ vim ~/.asdf/plugins/erlang/kerl-home/builds/asdf_26.0.2/otp_build_26.0.2.log
*** 2023年 8月 3日 星期四 11时52分57秒 CST - kerl build 26.0.2 ***
* KERL_BUILD_BACKEND=git
~/.asdf/plugins/erlang/kerl-home/builds/asdf_26.0.2/otp_src_26.0.2/configure  --cache-file=/dev/null CC=clang CPPFLAGS=-I/usr/local/opt/llvm/include\ -I/usr/local/opt/openssl\@3/include/ LDFLAGS=-L/usr/local/opt/llvm/lib\ -L/usr/local/opt/openssl\@3/lib --enable-darwin-64bit --with-ssl=/usr/local/opt/openssl@1.1
=== Running configure in /Users/cam/.asdf/plugins/erlang/kerl-home/builds/asdf_26.0.2/otp_src_26.0.2/erts ===
./configure 'CC=clang' 'CPPFLAGS=-I/usr/local/opt/llvm/include -I/usr/local/opt/openssl@3/include/' '--enable-darwin-64bit' '--with-ssl=/usr/local/opt/openssl@1.1' LDFLAGS='-L/usr/local/opt/llvm/lib -L/usr/local/opt/openssl@3/lib' --disable-option-checking --cache-file=/dev/null --srcdir="/Users/cam/.asdf/plugins/erlang/kerl-home/builds/asdf_26.0.2/otp_src_26.0.2/erts"
```