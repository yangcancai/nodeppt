title: Erlang程序入门教程
speaker: 飓风
plugins:
    - echarts

<slide class="bg-black-blue aligncenter" image="https://source.unsplash.com/C1HhAQrbykQ/ .dark">

# Erlang程序入门教程 {.text-landing.text-shadow}

By 飓风 {.text-intro}

[:fa-github: Github](https://github.com/yangcancai/nodeppt){.button.ghost}


<slide class="size-60 aligncenter">

## 环境和工具安装

```shell {.animated.fadeInUp}
# 安装包管理工具asf https://asdf-vm.com/guide/getting-started.html
$ git clone https://github.com/asdf-vm/asdf.git ~/.asdf --branch v0.10.2

# 增加下面配置到 ~/.bashrc 或者~/.zshrc
$ . $HOME/.asdf/asdf.sh

# 增加erlang插件
$ asdf plugin add erlang

# 安装erlang对应的版本
$ asdf install erlang 24.2.1

# 当前目录指定对应对应版本erlang
$ asdf local erlang 24.2.1

# 验证erlang openssl是否安装成功
$ erl
Erlang/OTP 24 [erts-12.2.1] [source] [64-bit] [smp:8:8] [ds:8:8:10] [async-threads:1] [jit]

Eshell V12.2.1  (abort with ^G)
1> crypto:start().
ok
2>
```

<slide class="aligncenter">

### rebar3安装

```shell 
$ wget https://github.com/erlang/rebar3/releases/download/3.19.0/rebar3 
$ mv rebar3 ~/.asdf/shims/rebar3
```

<slide :class="size-40 aligncenter">

## 创建项目

---
```shell
## 下载项目模版  
$ cd ~/.config/rebar3
git clone  https://github.com/yangcancai/rebar3_template.git templates

## 创建release项目
$ rebar3 new release proxy NAME=PROXY 
$ cd proxy && make
```

<slide :class="size-40 aligncenter">

## 数据类型
- Terms(以下所有数据的任意组合)
- Number(integer | float)
- Atom(常量)
- Binary(bytes)
- Reference
- Fun
- Port
- Pid
- Tuple
- Map
- List
- String
- Record
- Boolean
- Type Conversions

---

<slide :class="size-40 aligncenter">

## 模式匹配

---

<slide :class="size-40 aligncenter">

## 模块和函数

---

<slide :class="size-40 aligncenter">

## 类型和函数声明

---

