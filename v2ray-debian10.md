# 开启bbr

#### 修改系统变量 

echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf 

echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf

#### 保存生效 

sysctl -p

#### 查看内核是否开启bbr

sysctl net.ipv4.tcp_available_congestion_control

显示如下代表开启

\# sysctl net.ipv4.tcp_available_congestion_control

 net.ipv4.tcp_available_congestion_control = bbr cubic reno

#### 查看bbr是否启动 

lsmod | grep bbr



# 安装nginx

apt update

apt install nginx

#### 安装和配置ufw

apt install ufw 

ufw app list   #查看app list

ufw app info "OpenSSH" #查看app信息 

ufw allow OpenSSH #开启OpenSSH端口

ufw allow 443/tcp #开启443端口 

ufw allow "Nginx HTTP" #开启80端口

ufw enable #开启ufw

ufw status numbered #查看ufw状态

ufw allow 1024/tcp #开启v2端口 

systemctl enable nginx  #nginx开机启动

# 安装v2ray

apt-get update -y && apt install curl -y #安装curl

 bash <(curl -s -L https://git.io/v2ray.sh) #安装v2ray 

协议选websocket+tls

端口填写1024

域名填写要绑定的域名，其他默认配置就行，在 /etc/v2ray目录下会生成一个config.json文件，内容大概如下 

{

  "log": {

​    "access": "/var/log/v2ray/access.log",

​    "error": "/var/log/v2ray/error.log",

​    "loglevel": "warning"

  },

  "inbounds": [

​    {

​      "port": 1024,

​      "protocol": "vmess",

​      "settings": {

​        "clients": [

​          {

​            "id": "8368cd11-508b-4d39-b4d8-29a93f5228c4",

​            "level": 1,

​            "alterId": 0

​          }

​        ]

​      },

​      "streamSettings": {

​        "network": "ws"，

 "wsSettings": {

​    "path": "/mmws"

   }

​      },

​      "sniffing": {

​        "enabled": true,

​        "destOverride": [

​          "http",

​          "tls"

​        ]

​      }

​    }

​    //include_ss

​    //include_socks

​    //include_mtproto

​    //include_in_config

​    //

  ],

  "outbounds": [

​    {

​      "protocol": "freedom",

​      "settings": {

​        "domainStrategy": "UseIP"

​      },

​      "tag": "direct"

​    },

​    {

​      "protocol": "blackhole",

​      "settings": {},

​      "tag": "blocked"

​    },

​    {

​      "protocol": "mtproto",

​      "settings": {},

​      "tag": "tg-out"

​    }

​    //include_out_config

​    //

  ],

  "dns": {

​    "servers": [

​      "https+local://cloudflare-dns.com/dns-query",

​      "1.1.1.1",

​      "1.0.0.1",

​      "8.8.8.8",

​      "8.8.4.4",

​      "localhost"

​    ]

  },

  "routing": {

​    "domainStrategy": "IPOnDemand",

​    "rules": [

​      {

​        "type": "field",

​        "ip": [

​          "0.0.0.0/8",

​          "10.0.0.0/8",

​          "100.64.0.0/10",

​          "127.0.0.0/8",

​          "169.254.0.0/16",

​          "172.16.0.0/12",

​          "192.0.0.0/24",

​          "192.0.2.0/24",

​          "192.168.0.0/16",

​          "198.18.0.0/15",

​          "198.51.100.0/24",

​          "203.0.113.0/24",

​          "::1/128",

​          "fc00::/7",

​          "fe80::/10"

​        ],

​        "outboundTag": "blocked"

​      },

​      {

​        "type": "field",

​        "inboundTag": ["tg-in"],

​        "outboundTag": "tg-out"

​      }

​      ,

​      {

​        "type": "field",

​        "domain": [

​          "domain:epochtimes.com",

​          "domain:epochtimes.com.tw",

​          "domain:epochtimes.fr",

​          "domain:epochtimes.de",

​          "domain:epochtimes.jp",

​          "domain:epochtimes.ru",

​          "domain:epochtimes.co.il",

​          "domain:epochtimes.co.kr",

​          "domain:epochtimes-romania.com",

​          "domain:erabaru.net",

​          "domain:lagranepoca.com",

​          "domain:theepochtimes.com",

​          "domain:ntdtv.com",

​          "domain:ntd.tv",

​          "domain:ntdtv-dc.com",

​          "domain:ntdtv.com.tw",

​          "domain:minghui.org",

​          "domain:renminbao.com",

​          "domain:dafahao.com",

​          "domain:dongtaiwang.com",

​          "domain:falundafa.org",

​          "domain:wujieliulan.com",

​          "domain:ninecommentaries.com",

​          "domain:shenyun.com"

​        ],

​        "outboundTag": "blocked"

​      }      ,

​        {

​          "type": "field",

​          "protocol": [

​            "bittorrent"

​          ],

​          "outboundTag": "blocked"

​        }

​      //include_ban_ad

​      //include_rules

​      //

​    ]

  },

  "transport": {

​    "kcpSettings": {

​      "uplinkCapacity": 100,

​      "downlinkCapacity": 100,

​      "congestion": true

​    }

  }

}

# 配置nginx

cd /etc/nginx/site-enables

 vim cf.conf

配置信息如下 

server {

  listen 443 ssl http2;

  server_name 绑定的域名;

  index index.html index.htm index.php;

  root /data/wwwroot/;



  ssl_certificate /root/cfkey/cfcer.pem;

  ssl_certificate_key /root/cfkey/cfkey.key;

  ssl_session_cache shared:SSL:10m;

  ssl_session_timeout 10m;

  ssl_protocols TLSv1.2 TLSv1.3;

  ssl_prefer_server_ciphers on;

  ssl_ciphers EECDH+AESGCM:EDH+AESGCM;



  



  location /mmws {

  proxy_redirect off;

  proxy_http_version 1.1;

  proxy_set_header Upgrade $http_upgrade;

  proxy_set_header Connection "upgrade";

  proxy_set_header Host $http_host;

  proxy_set_header X-Real-IP $remote_addr;

  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;



  set $is_v2ray 0;

  if ($http_upgrade = "websocket") {

​    set $is_v2ray 1;

  }



  if ($is_v2ray = 1) {

​    \# 仅当请求为 WebSocket 时才反代到 V2Ray

​    proxy_pass http://127.0.0.1:1024;

  }



  if ($is_v2ray = 0) {

​    \# 否则显示正常网页

​    rewrite ^/(.*)$ /mask-page last;

  }

  }

}

