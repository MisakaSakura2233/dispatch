# dispatch

一种Rust语言的客户端负载均衡器，可实现带宽叠加、带宽聚合。其可用于macOS, Windows以及Linux平台。

这是基于 [dispatch-proxy](https://github.com/alexkirsz/dispatch-proxy)的rust重写版本。

## 使用说明

### 关于适用环境

原作者仅在Macos上进行了测试，但是在Windows 11 workstation 版本中也能正常运行

### 前提条件（windows平台）

你需要 Rust 1.51.0 或者以上版本，原作者推荐使用[rustup](https://rustup.rs/)进行rust最新版本安装

```
cargo install dispatch-proxy
```

### 开始配置

在windows平台中，默认是不能同时使用wifi与以太网连接，因此需要进行组策略的更改（改接口跃点数的不可能，原理请读者自证）以下是更改组策略方式完成同时使用

方法如下：

1.打开运行窗口, 输入快捷命令
```
gpedit.msc Q
```
然后回车

2.在本地计算机策略→计算机配置→管理模块→网络→Windows连接管理器→最小化到internet或Windows域的同时连接数→编辑（需右键）一已启用→最小化策略选项→0=允许同时连接。

若在（最小化到internet或Windows域的同时连接数）中显示为默认的未配置，只需要改为已启用，然后选择0=允许同时连接。

### 开始使用
可以自由配置，以下给出几个举例
```
$ dispatch start 10.0.0.0 fdaa:bbcc:ddee:0:1:2:3:4
```
```
$ dispatch start  --ip 127.0.0.1 --port 10806 192.168.110.101@7 192.168.110.102@3
```
```
$ dispatch start 10.0.0.0 10.255.255.255 172.31.255.255 fdaa:bbcc:ddee:0:1:2:3:4
```

## 其适用情况
让人忍无可忍的垃圾校园网络（提供WIFI，以太网连接）
想通过纯软件方式将网络叠加与加权的负载均衡
可以通过[blueskyxn](https://github.com/BlueSkyXN)的一个方法进行套娃使用，以下为引用：

1.V2rayN作为出口，需要做好DNS等设置，如果有异常需要重启服务
2.Netch作为进程代理控制程序，配置并使用Dispatch提供的S5代理，对V2rayN的二进制程序启用（包括内核，比如xray.exe、sing-box.exe）
3.Dispatch自行选择端口和可用的本地IP，作为上联源，以及配置权重，默认是均等连接，然后默认端口是1080，不可冲突端口
（套娃方式不一定适用于自身情况，按需求适用）

## 解释用法

```
Usage: dispatch [OPTIONS] <COMMAND>

Commands:
  list   Lists all available network interfaces
  start  Starts the SOCKS proxy server
  help   Print this message or the help of the given subcommand(s)

Options:
  -d, --debug    Write debug logs to stdout instead of a file
  -h, --help     Print help
  -V, --version  Print version
```

```
$ dispatch start -h
Starts the SOCKS proxy server

Usage: dispatch start [OPTIONS] <ADDRESSES>...

Arguments:
  <ADDRESSES>...  The network interface IP addresses to dispatch to, in the form of <address>[/priority]

Options:
      --ip <IP>      Which IP to accept connections from [default: 127.0.0.1]
      --port <PORT>  Which port to listen to for connections [default: 1080]
  -h, --help         Print help
```

## 以下是原作者的举例

```
$ dispatch list
```

Lists all available network interfaces.

```
$ dispatch start 10.0.0.0 fdaa:bbcc:ddee:0:1:2:3:4
```

Dispatch incoming connections to local addresses `10.0.0.0` and `fdaa:bbcc:ddee:0:1:2:3:4`.

```
$ dispatch start 10.0.0.0/7 10.0.0.1/3
```

Dispatch incoming connections to `10.0.0.0` 7 times out of 10 and to `10.0.0.1` 3 times out of 10.

## 运行原理

Whenever the SOCKS proxy server receives an connection request to an address or domain, it selects one of the provided local addresses using the [Weighted Round Robin](https://en.wikipedia.org/wiki/Weighted_round_robin) algorithm. All further connection traffic will then go through the interface corresponding to the selected local address.

**Beware:** If the requested address or domain resolves to an IPv4 (resp. IPv6) address, an IPv4 (resp. IPv6) local address must be provided.

#### License

<sup>
Licensed under either of <a href="LICENSE-APACHE">Apache License, Version
2.0</a> or <a href="LICENSE-MIT">MIT license</a> at your option.
</sup>

<br>

<sub>
Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in this crate by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.
</sub>
