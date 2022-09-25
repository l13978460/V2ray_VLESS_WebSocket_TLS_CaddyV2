# 2022-9-19 
Hax / Woiden 站长时不时的把机器人验证调得很难，于是续期非常容易失败。
那么把搭梯子的脚本简单化
```
apt update && apt install -y curl && bash <(curl -L https://github.com/l13978460/V2ray_VLESS_WebSocket_TLS_CaddyV2/raw/main/install.sh) 你的域名 6 你的UUID 你的path
```
跑这条命令之前，把CDN关闭。跑完之后再把CDN打开。
这样搭出来的梯子，你的翻墙客户端节点信息不用改。

# 2022-9-8
临时地，本脚本指定安装V2ray v4.45.2 (v5之前的最后一个v4)

相关信息
https://github.com/v2fly/fhs-install-v2ray/issues/243

# 说明
手机走数据流量有IPv6，用你的手机流量访问 http://test-ipv6.com/ 测试一下。
要有个SSH工具。iOS可以用 Termius 安卓可以用  JuiceSSH (官方网站changelog)

创建访问 https://hax.co.id/ 点击左上角 "三" - Register
 点击机器人的名字，发送 /getid 给机器人，就会得到你的ID，将其填到此页面内。点击"Submit"
接收验证码，填写到页面中，输入你的账户密码。点击"Submit"。登录账户（login）输入账户密码，机器人验证。点击 "三" - "VPS" - "Create VPS"
数据中心随便选，不是OPENVZ的就行，操作系统推荐 Debian 11，root密码自己定一个，VPS目的随便选，勾上一堆"我同意"，再通过一下人机验证。点击 "CREATE VPS"
过几分钟去看 VPS - VPS Info。把你的 VPS 的 IPv6 地址记下来。

申请Hax提供的免费域名，点击 DNS Pointing CF。Domain Name 随便选（不成功就换一个），CF Proxy - No，DNS Name - 数字字母组合随便弄一个，IPv6 Address - 你的VPS的IPv6地址，通过人机验证，点击 Create DNS。记下域名

手机SSH登录，打开 Termius，进入 Host。右上角的 "+" 号，new Host。Alias随便填，Hostname 填 VPS 的 IPv6 地址，Username 填 root，Password 填 root 密码 (Create VPS那一步填的密码)，点击 "Save".打开保存的host按指示走

这个一键脚本超级简单。有效语句11行(其中BBR 5行, 安装V2Ray 1行, 安装Caddy 5行)+Caddy配置文件15行(其中你需要修改4行)+V2Ray配置文件89行(其中你需要修改2行), 其它都是用来检验小白输入错误参数或者搭建条件不满足的。


# 一键执行
```
apt update
apt install -y curl
```
```
bash <(curl -L https://github.com/l13978460/V2ray_VLESS_WebSocket_TLS_CaddyV2/raw/main/install.sh)
```
总共三条命令，最后按提示输入，域名填刚申请的，ipv6那里选6，其它可以默认，最终得到一个vless://链接。利用warp给纯IPv6的小鸡添加IPv4对外访问的能力，回车继续直到结束，或者输入命令 
bash <(curl -fsSL git.io/warp.sh) 4

回到前面添加域名的那里dns-pointing-cf（检查登录状态），先删除CF Proxy为No的那条DNS，再申请一个CF Proxy为Yes的DNS，注意只有CF Proxy为Yes，其它与之前的保持一致。
（结束）搬运这些字只是为自己方便看不用到处翻，一次成功。其它问题看https://zelikk.blogspot.com/（已墙）


# 打开BBR
```
sed -i '/net.ipv4.tcp_congestion_control/d' /etc/sysctl.conf
sed -i '/net.core.default_qdisc/d' /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control = bbr" >>/etc/sysctl.conf
echo "net.core.default_qdisc = fq" >>/etc/sysctl.conf
sysctl -p >/dev/null 2>&1
```

# 安装V2ray最新版本
source: https://github.com/v2fly/fhs-install-v2ray
```
bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
```

# 安装CaddyV2最新版本
source: https://caddyserver.com/docs/install#debian-ubuntu-raspbian=

```
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

如果已经装过了Caddy, 重装的时候脚本会问你
```
File '/usr/share/keyrings/caddy-stable-archive-keyring.gpg' exists. Overwrite? (y/N)
```
输入 y 回车。

# 配置 /usr/local/etc/v2ray/config.json
```
{ // VLESS + WebSocket + TLS
    "log": {
        "access": "/var/log/v2ray/access.log",
        "error": "/var/log/v2ray/error.log",
        "loglevel": "warning"
    },
    "inbounds": [
        {
            "listen": "127.0.0.1",        
            "port": 你的v2ray内部端口,             // ***改这里
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "你的v2rayID",             // ***改这里
                        "level": 1,
                        "alterId": 0
                    }
                ],
                "decryption": "none"
            },
            "streamSettings": {
                "network": "ws"
            },
            "sniffing": {
                "enabled": true,
                "destOverride": [
                    "http",
                    "tls"
                ]
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom",
            "settings": {
                "domainStrategy": "UseIP"
            },
            "tag": "direct"
        },
        {
            "protocol": "blackhole",
            "settings": {},
            "tag": "blocked"
        }
    ],
    "dns": {
        "servers": [
            "https+local://8.8.8.8/dns-query",
            "8.8.8.8",
            "1.1.1.1",
            "localhost"
        ]
    },
    "routing": {
        "domainStrategy": "IPOnDemand",
        "rules": [
            {
                "type": "field",
                "ip": [
                    "0.0.0.0/8",
                    "10.0.0.0/8",
                    "100.64.0.0/10",
                    "127.0.0.0/8",
                    "169.254.0.0/16",
                    "172.16.0.0/12",
                    "192.0.0.0/24",
                    "192.0.2.0/24",
                    "192.168.0.0/16",
                    "198.18.0.0/15",
                    "198.51.100.0/24",
                    "203.0.113.0/24",
                    "::1/128",
                    "fc00::/7",
                    "fe80::/10"
                ],
                "outboundTag": "blocked"
            },
            {
                "type": "field",
                "protocol": [
                    "bittorrent"
                ],
                "outboundTag": "blocked"
            }
        ]
    }
}
```

# 配置 /etc/caddy/Caddyfile
```
你的域名     # 改这里
{
    tls Y3JhenlwZWFjZQ@gmail.com
    encode gzip

    handle_path /分流path {     # 改这里
        reverse_proxy localhost:你的v2ray内部端口     # 改这里
    }
    handle {
        reverse_proxy https://你反代伪装的网站 {     # 改这里
            trusted_proxies 0.0.0.0/0
            header_up Host {upstream_hostport}
        }
    }
}
```

如果想多用户使用，可以通过多path的方式
```
你的域名     # 改这里
{
    tls Y3JhenlwZWFjZQ@gmail.com
    encode gzip

@ws_path {
    path /分流path1     # 改这里
    path /分流path2     # 改这里
    path /分流path3     # 改这里
}

    handle @ws_path {
        uri path_regexp /.* /
        reverse_proxy localhost:你的v2ray内部端口     # 改这里
    }
    handle {
        reverse_proxy https://你反代伪装的网站 {     # 改这里
            trusted_proxies 0.0.0.0/0
            header_up Host {upstream_hostport}
        }
    }
}
```
可参考视频 https://www.youtube.com/watch?v=bfZh_eaYJLE&t=220s

# 如果是 IPv6 only 的小鸡，用 WARP 添加 IPv4 出站能力
```
bash <(curl -L git.io/warp.sh) 4
```

# Uninstall
```
bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh) --remove
rm /etc/apt/sources.list.d/caddy-stable.list
apt remove -y caddy
```

# 私货
对于喜欢V2rayN PAC模式的朋友，欢迎使用支持VLESS链接导入功能的 [v2rayN-3.29-VLESS](https://github.com/crazypeace/v2rayN-3.29-VLESS)
![v2rayN_2022-07-20_22-02-43](https://user-images.githubusercontent.com/665889/180002616-c2c6da3c-78b0-4f46-8fa9-34021590646f.png)

# 带参数执行
如果你已经很熟悉了, 安装过程中的参数都确认没问题. 可以带参数使用本脚本, 跳过脚本中的各种校验.
```
bash <(curl -L https://github.com/crazypeace/V2ray_VLESS_WebSocket_TLS_CaddyV2/raw/main/install.sh) <domain> [netstack] [UUID] [path]
```
其中

`domain`      你的域名

`netstask`    6 表示 IPv6入站, 最后会安装WARP获得IPv4出站

`UUID` 你的UUID

`path` 你的path，如果不输入，会从UUID自动生成

例如
```
bash <(curl -L https://github.com/crazypeace/V2ray_VLESS_WebSocket_TLS_CaddyV2/raw/main/install.sh) abc.mydomain.com
bash <(curl -L https://github.com/crazypeace/V2ray_VLESS_WebSocket_TLS_CaddyV2/raw/main/install.sh) abccba.ipv6d.my.id 6
bash <(curl -L https://github.com/crazypeace/V2ray_VLESS_WebSocket_TLS_CaddyV2/raw/main/install.sh) abccba.ipv6d.my.id 6 486572e1-11d5-4e93-a41d-d4b9775870bd
bash <(curl -L https://github.com/crazypeace/V2ray_VLESS_WebSocket_TLS_CaddyV2/raw/main/install.sh) abccba.ipv6d.my.id 6 486572e1-11d5-4e93-a41d-d4b9775870bd somepath
```

## 用你的STAR告诉我这个Repo对你有用 Welcome STARs! :)

[![Stargazers over time](https://starchart.cc/crazypeace/V2ray_VLESS_WebSocket_TLS_CaddyV2.svg)](https://starchart.cc/crazypeace/V2ray_VLESS_WebSocket_TLS_CaddyV2)
