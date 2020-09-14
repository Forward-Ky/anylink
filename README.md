# AnyLink

AnyLink 是一个企业级远程办公vpn软件，可以支持多人同时在线使用。

## Introduction

AnyLink 基于 [ietf-openconnect](https://tools.ietf.org/html/draft-mavrogiannopoulos-openconnect-02) 协议开发，并且借鉴了 [ocserv](http://ocserv.gitlab.io/www/index.html) 的开发思路，使其可以同时兼容 AnyConnect 客户端。

AnyLink 使用TLS/DTLS进行数据加密，因此需要RSA或ECC证书，可以通过 Let's Encrypt 和 TrustAsia 申请免费的SSL证书。

AnyLink 服务端仅在CentOS7测试通过，如需要安装在其他系统，需要服务端支持tun/tap功能、ip设置命令。


## Installation

```
git clone https://github.com/bjdgyc/anylink.git
cd anylink
go build -o anylink -ldflags "-X main.COMMIT_ID=`git rev-parse HEAD`"
#注意使用root权限运行
sudo ./anylink -conf="conf/server.toml"
```

## Feature

- [x] IP分配(实现IP、MAC映射信息的持久化)
- [x] TLS-TCP通道
- [x] 兼容AnyConnect
- [x] 基于tun设备的nat访问模式
- [x] 基于tap设备的桥接访问模式
- [x] 多用户支持
- [x] 支持 [proxy protocol v1](http://www.haproxy.org/download/2.2/doc/proxy-protocol.txt) 协议

- [ ] 用户组支持
- [ ] TOTP令牌支持
- [ ] 流量控制
- [ ] 访问权限管理
- [ ] 后台管理界面
- [ ] DTLS-UDP通道


## Config

默认配置文件内有详细的注释，根据注释填写配置即可。

- [conf/server.toml](https://github.com/bjdgyc/anylink/blob/master/conf/server.toml)

## Setting

网络模式选择，需要配置 `link_mode` 参数，如 `link_mode="tun"`,`link_mode="tap"` 两种参数。
不同的参数需要对服务器做相应的设置。

建议优先选择tun模式，因客户端传输的是IP层数据，无须进行数据转换。
tap模式是在用户态做的链路层到IP层的数据互相转换，性能会有所下降。

### tun设置

1. 开启服务器转发
    ```
    # flie: /etc/sysctl.conf
    net.ipv4.ip_forward = 1

    #执行如下命令
    sysctl -w net.ipv4.ip_forward=1
    ```

2. 设置nat转发规则
    ```
    # eth0为服务器内网网卡
    iptables -t nat -A POSTROUTING -s 192.168.10.0/255.255.255.0 -o eth0 -j MASQUERADE
    ```
   
3. 使用AnyConnect客户端连接即可


### tap设置
    
1. 创建桥接网卡
    ```
    注意 server.toml 的ip参数，需要与 bridge.sh 的配置参数一致
    ```
   
2. 修改 bridge-init.sh 内的参数
    ```
    # file: ./bridge.sh
    eth="eth0"
    eth_ip="192.168.1.4"
    eth_netmask="255.255.255.0"
    eth_broadcast="192.168.1.255"
    eth_gateway="192.168.1.1"
    ```
    
3. 执行 bridge.sh 文件
    ```
    sh bridge.sh
    ```
    


## License

本项目采用 MIT 开源授权许可证，完整的授权说明已放置在 LICENSE 文件中。







