# v2ray configuration example

## With kcp and `wechat-video` encapsulation.
```json
{
    "inbound": {
        "port": 11111,
        "protocol": "vmess",
        "settings": {
            "clients": [
                {
                    "id": "Your UUID",
                    "alterId": 64,
                    "level": 1
                }
            ]
        },
        "streamSettings": {
            "network": "mkcp",
            "kcpSettings": {
                "mtu": 1350,
                "tti": 50,
                "uplinkCapacity": 100,
                "downlinkCapacity": 100,
                "congestion": false,
                "readBufferSize": 2,
                "writeBufferSize": 2,
                "header": {
                    "type": "wechat-video"
                }
            }
        }
    },
    "outbound": {
        "protocol": "freedom",
        "settings": {}
    }
}
```
