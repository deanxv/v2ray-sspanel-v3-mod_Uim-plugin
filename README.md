# 2024.0122
解决docker版本过低的问题。


感恩原作者rico辛苦付出
本人仅做备份和后续维护
caddy镜像更新支持tls1.3

# v2ray-sspanel-v3-mod_Uim-plugin


## Thanks
1. 感恩的 [ColetteContreras's repo](https://github.com/ColetteContreras/v2ray-ssrpanel-plugin). 让我一个go小白有了下手地。主要起始框架来源于这里
2. 感恩 [eycorsican](https://github.com/eycorsican) 在v2ray-core [issue](https://github.com/v2ray/v2ray-core/issues/1514), 促成了go版本提上日程


# 划重点
1. 用户务必保证，host 务必填写没有被墙的地址
2. 已经适配了中转，必须用我自己维护的[panel](https://github.com/rico93/ss-panel-v3-mod_Uim)


## 项目状态

支持 [ss-panel-v3-mod_Uim](https://github.com/NimaQu/ss-panel-v3-mod_Uim) 的 webapi。 目前自己也尝试维护了一个版本

目前只适配了流量记录、服务器是否在线、在线人数,在线ip上报、负载、中转，后端根据前端的设定自动调用 API 增加用户。

v2ray 后端 kcp、tcp、ws 都是多用户共用一个端口。

也可作为 ss 后端一个用户一个端口。

## 已知 Bug

## 作为 ss 后端

面板配置是节点类型为 Shadowsocks，普通端口。

加密方式只支持：

- [x] aes-256-cfb
- [x] aes-128-cfb
- [x] chacha20
- [x] chacha20-ietf
- [x] aes-256-gcm
- [x] aes-128-gcm
- [x] chacha20-poly1305 或称 chacha20-ietf-poly1305

## 作为 V2ray 后端

这里面板设置是节点类型v2ray, 普通端口。 v2ray的API接口默认是2333

支持 tcp,kcp、ws+(tls 由镜像 Caddy或者ngnix 提供,默认是443接口哦)。或者自己调整。

[面板设置说明 主要是这个](https://github.com/NimaQu/ss-panel-v3-mod_Uim/wiki/v2ray-%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B)

~~~
没有CDN的域名或者ip;端口（外部链接的);AlterId;协议层;;额外参数(path=/v2ray|host=xxxx.win|inside_port=10550这个端口内部监听))

// ws 示例
xxxxx.com;10550;16;ws;;path=/v2ray|host=oxxxx.com

// ws + tls (Caddy 提供)
xxxxx.com;0;16;tls;ws;path=/v2ray|host=oxxxx.com|inside_port=10550
xxxxx.com;;16;tls;ws;path=/v2ray|host=oxxxx.com|inside_port=10550



// nat🐔 ws 示例
xxxxx.com;11120;16;ws;;path=/v2ray|host=oxxxx.com

// nat🐔 ws + tls (Caddy 提供)
xxxxx.com;0;16;tls;ws;path=/v2ray|host=oxxxx.com|inside_port=10550|outside_port=11120
xxxxx.com;;16;tls;ws;path=/v2ray|host=oxxxx.com|inside_port=10550|outside_port=11120
~~~

目前的逻辑是

- 如果为外部链接的端口是0或者不填，则默认监听本地127.0.0.1:inside_port
- 如果外部端口设定不是 0或者空，则监听 0.0.0.0:外部设定端口，此端口为所有用户的单端口，此时 inside_port 弃用。
- 默认使用 Caddy 镜像来提供 tls，控制代码不会生成 tls 相关的配置。Caddyfile 可以在Docker/Caddy_V2ray文件夹里面找到。
- Nat🐔，如果要用ws+tls，则需要使用outside_port=xxx，php后端会生成订阅时候，使用outside_port覆盖port部分。 outside_port是内部映射端口，
 建议内网和外网的两个端口数值一致。

tcp 配置：

~~~
xxxxx.com;非0;16;tcp;;
~~~

kcp 支持所有 v2ray 的 type：

- none: 默认值，不进行伪装，发送的数据是没有特征的数据包。

~~~
xxxxx.com;非0;16;kcp;noop;
~~~

- srtp: 伪装成 SRTP 数据包，会被识别为视频通话数据（如 FaceTime）。

~~~
xxxxx.com;非0;16;kcp;srtp;
~~~

- utp: 伪装成 uTP 数据包，会被识别为 BT 下载数据。

~~~
xxxxx.com;非0;16;kcp;utp;
~~~

- wechat-video: 伪装成微信视频通话的数据包。

~~~
xxxxx.com;非0;16;kcp;wechat-video;
~~~

- dtls: 伪装成 DTLS 1.2 数据包。

~~~
xxxxx.com;非0;16;kcp;dtls;
~~~

- wireguard: 伪装成 WireGuard 数据包(并不是真正的 WireGuard 协议) 。

~~~
xxxxx.com;非0;16;kcp;wireguard;
~~~

### [可选] 安装 BBR

看 [Rat的](https://www.moerats.com/archives/387/)
OpenVZ 看这里 [南琴浪](https://github.com/tcp-nanqinlang/wiki/wiki/lkl-haproxy)

~~~
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh
~~~

Ubuntu 18.04 魔改 BBR 暂时有点问题，可使用以下命令安装：

~~~
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh"
apt install make gcc -y
sed -i 's#/usr/bin/gcc-4.9#/usr/bin/gcc#g' '/root/tcp.sh'
chmod +x tcp.sh && ./tcp.sh
~~~
### [可选] 增加swap
整数是M
~~~
wget https://www.moerats.com/usr/shell/swap.sh && bash swap.sh
~~~

### [推荐] 脚本部署

#### Docker-compose 安装 
这里一直保持最新版
~~~
mkdir v2ray-agent  &&  \
cd v2ray-agent && \
curl https://raw.githubusercontent.com/deanxv/v2ray-sspanel-v3-mod_Uim-plugin/master/install.sh -o install.sh && \
chmod +x install.sh && \
bash install.sh
~~~

##### 安装caddy

一键安装 caddy 和cf ddns tls插件

~~~
curl https://getcaddy.com | bash -s dyndns,tls.dns.cloudflare
~~~

Caddyfile 

自行修改，或者设置对应环境变量

~~~
{$V2RAY_DOMAIN}:{$V2RAY_OUTSIDE_PORT}
{
  root /srv/www
  log ./caddy.log
  proxy {$V2RAY_PATH} 127.0.0.1:{$V2RAY_PORT} {
    websocket
    header_upstream -Origin
  }
  gzip
  tls {$V2RAY_EMAIL} {
    protocols tls1.0 tls1.2
    # remove comment if u want to use cloudflare ddns
    # dns cloudflare
  }
}
~~~
