# ShadowsocksX-NG Gost Plugin

ShadowsocksX-NG 的 gost 插件脚本，方便在 ShadowsocksX-NG 中使用 gost 

## 原由

自从查资料上网工具换成 [gost](https://github.com/ginuerzh/gost) 之后，由于 MacOS 上没有 gost 专用的智能代理（也就是该翻的时候翻，不用翻的时候不翻）桌面客户端，所以需要用 gost 在本地把 https 代理转成 ss 后再继续使用 ShadowsocksX-NG。 虽然可以用 launchctl 启动一个 gost 后台服务，但是用起来还是不太方便。

看了下 ShadowsocksX-NG 是如何工作的： ShadowsocksX-NG 会在本地启动 ss-local 进程跑一个 socks5 服务，而且 ShadowsocksX-NG 实现智能代理的逻辑与 ss-local 并没有太多的关系，所以只要在想办法本地能提供一个 socks5 服务就够了。 

于是写了以下脚本替换了 ShadowsocksX-NG 安装目录下的 ss-local(替换之前要备份一下这个文件)

```bash
#!/bin/sh -

#/opt/gost/gost -L=sock5://:1086 -F=wss://username:password@1.1.1.1:443

# Use gost.json
#{
#    "Retries": 3,
#    "Debug": false,
#    "ServeNodes": [
#        "socks5://127.0.0.1:1086"
#    ],
#    "ChainNodes": [
#        "wss://username:password@1.1.1.1:443"
#    ]
#}
/opt/gost/gost -C /opt/gost/gost.json
```

替换之后，ShadowsocksX-NG 可以正常工作，但是这样 ShadowsocksX-NG 的其它设置就无法工作了，所以用 python3 把功能加强了一下，编写了这个脚本。

## 安装插件前的准备

1. 安装好 [ShadowsocksX-NG](https://github.com/shadowsocks/ShadowsocksX-NG/releases/download/v1.9.4/ShadowsocksX-NG.1.9.4.zip) 并至少启动过一次
2. 系统已经安装好 python3

## 手动安装插件

手动安装过程包括以下几个步骤，主要是文件的替换：

1. 下载 [gost](https://github.com/ginuerzh/gost/releases/download/v2.11.1/gost-darwin-amd64-2.11.1.gz) 并解压到目录 `"${HOME}/Library/Application Support/ShadowsocksX-NG/gost"`， 确保 `"${HOME}/Library/Application Support/ShadowsocksX-NG/gost"` 目录下可执行文件名称为 `gost`
2. 备份 `/Applications/ShadowsocksX-NG.app/Contents/Resources/ss-local` 为 `/Applications/ShadowsocksX-NG.app/Contents/Resources/real-ss-local` 
3. 用 `https://raw.githubusercontent.com/lewangdev/gost-ss-local/master/ss-local` 替换 `/Applications/ShadowsocksX-NG.app/Contents/Resources/ss-local` 
4. 退出 ShadowsocksX-NG 应用，再打开即可正常使用

## 通过自动安装脚本安装插件

把上面的手动安装的过程变成自动安装的脚本

```bash
curl -L https://github.com/lewangdev/ShadowsocksX-NG-GostPlugin/raw/master/gost-plugin-installer | bash
```

## 设置

ShadowsocksX-NG 客户端的配置并不能与 gost 的配置对应上，gost-ss-local 使用了 Address，Port，Password，Plugin，Plugin Opts 来设置 gost。假设 gost 在服务器 1.2.3.4 上是这样启动的：

```
gost -L wss://username:password@:443
```

那么在 Plugin 使用 gost 之后，各参数设置如下: 

0. Plugin，如果希望使用 gost，那么 Plugin 需要填写 gost，例如 `gost`，不填或填其它内容，则与 ShadowsocksX-NG 原行为一致
1. Address, 表示 gost 的服务器地址，可以是 IP 或域名, 例如填写 `1.2.3.4`
2. Port, 表示 gost 的端口, 例如填写 `443`
3. Password, 由于 ShadowsocksX-NG 不能设置用户名，密码这里需要填写 gost 的用户名和密码，格式为 `username:password`, 例如填写 `username:password`
4. Plugin Opts, 如果填写了插件参数，则前 1-3 的设置无效，并且会把 Plugin Opts 填写的内容直接全部传给 gost 命令。


<div align="center">
  <img width="60%" src="https://raw.githubusercontent.com/lewangdev/picb0/master/shadowsocksX-NG-GostPlugin-1.png">
</div>


1-3 的设置要求 gost 服务器端为 https 代理，如果为其它类型的代理，可以通过设置 Plugin Opts 的参数来设置，例如与前面 1-3 等价的 Plugin Opts 配置为 `-L socks5://127.0.0.1:1086 -F wss://username:password@1.2.3.4:443`


<div align="center">
  <img width="60%" src="https://raw.githubusercontent.com/lewangdev/picb0/master/shadowsocksX-NG-GostPlugin-2.png">
</div>


## 说明

目前脚本只测试了 gost，对于 [SIP003 Plugin](https://github.com/shadowsocks/ShadowsocksX-NG/wiki/SIP003-Plugin) 是否影响没有做过测试，如果有问题，欢迎前往[代码仓库](https://github.com/gost-x/ShadowsocksX-NG-GostPlugin)提 [Issues](https://github.com/gost-x/ShadowsocksX-NG-GostPlugin/issues)。

