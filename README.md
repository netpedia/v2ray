# v2ray
## V2Ray Server Installation

### V2Ray in Docker
```sh
docker run -d --net iteungwebnet --ip 172.18.0.39 --name v2ray -v /home/docker/v2ray:/etc/v2fly -p 10086:10086 v2fly/v2fly-core run -c /etc/v2fly/config.json
```

config.json
```json
{
  "log": {
    "loglevel": "warning",
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log"
  },
  "inbounds": [
    {
      "port": 10086,
      "listen":"0.0.0.0",
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30811",
            "alterId": 64
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
        "path": "/ray"
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```
setup time zone and
restart docker
cek logs
```sh
docker ps -a | grep v2ray
docker restart v2ray
docker logs v2ray
```
### Nginx setup

```sh
server {
  listen 80;
  server_name    example.com;

  index index.html;
  root /usr/share/nginx/html/;

  access_log /var/log/nginx/v2ray.access;
  error_log /var/log/nginx/v2ray.error;

    location /ray { # Consistent with the path of V2Ray configuration
      if ($http_upgrade != "websocket") { # Return 404 error when WebSocket upgrading negotiate failed
          return 404;
      }
      proxy_redirect off;
      proxy_pass http://127.0.0.1:10086; # Assume WebSocket is listening at localhost on port of 10000
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
      # Show real IP in v2ray access.log
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### BBR For Slow Speed
In Docker Host and Nginx Host

```sh
wget -N --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && bash bbr.sh
```
or

```sh
echo "net.core.default_qdisc=fq" | sudo tee -a /etc/sysctl.d/60-custom.conf

echo "net.ipv4.tcp_congestion_control=bbr" | sudo tee -a /etc/sysctl.d/60-custom.conf

sudo sysctl -p /etc/sysctl.d/60-custom.conf

```

## V2Ray Client

Download client : https://github.com/v2fly/v2ray-core/releases

edit json
```json
{
  "inbounds": [
    {
      "port": 1090,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      },
      "settings": {
        "auth": "noauth",
        "udp": false
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "example.com",
            "port": 443,
            "users": [
              {
                "id": "b831381d-6324-4d53-ad4f-8cda48b30811",
                "alterId": 0
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "wsSettings": {
          "path": "/ray"
        }
      }
    }
  ]
}
```

enter CMD type 
```sh
v2ray.exe run
```

Setup SOCKS 5 Proxy in your browser

![image](https://user-images.githubusercontent.com/11188109/235120455-bef7c819-21e1-4867-ad0d-289731e94a97.png)
