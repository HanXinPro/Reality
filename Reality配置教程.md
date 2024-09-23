# *Reality*配置教程

## sing-box

### *RealiTLScanner*扫描

#### 下载扫描工具

```
wget https://github.com/XTLS/RealiTLScanner/releases/download/v0.2.1/RealiTLScanner-linux-64
mv ./RealiTLScanner-linux-64 ./RealiTLScanner  
chmod +x ./RealiTLScanner
```

> 可以使用Claude等AI帮助理解，可以将以下示例发给AI：

*Scan a specific IP, IP CIDR or domain:*  
`./RealiTLScanner -addr 1.2.3.4`  
*Note: infinity mode will be enabled automatically if `addr` is an IP or domain*  

*Show verbose output, including failed scans and infeasible targets:*  
`./RealiTLScanner -addr 1.2.3.0/24 -v`

*Save results to a file, default: out.csv*  
`./RealiTLScanner -addr www.microsoft.com -out file.csv`

*Set a thread count, default: 1*  
`./RealiTLScanner -addr wiki.ubuntu.com -thread 10`

*Set a timeout for each scan, default: 10 (seconds)*
`./RealiTLScanner -addr 107.172.1.1/16 -timeout 5`

> 可以要求AI根据上述示例扫描`Your_server_IP`的邻居，可以要求扫描5000个左右的“邻居”，`100`线程，`3`秒超时，文件名`Your_filename.csv`

eg:扫描`100.100.100.100`的邻居,扫描数量8000个，`100`线程，`3`秒超时，文件名`100.csv`

~~啥也不懂直接`./RealiTLScanner Your_server_IP`~~  

> 好了，下一步

---  

### 安装sing-box

> 如果你选择了sing-box内核，那就先安装sing-box，使用下面的代码

在终端中输入  
```bash <(curl -fsSL https://sing-box.app/deb-install.sh)```
此时，sing-box已经被安装到Debian中  
打开sing-box配置文件目录  
```cd /etc/sing-box```  
编辑`config.json`双击打开  

```
{
    "inbounds": [
        {
            "type": "vless",
            "listen": "::",
            "listen_port": 443,
            "users": [
                {
                    "uuid": "", // 执行sing-box generate uuid 生成
                    "flow": "xtls-rprx-vision"
                }
            ],
            "tls": {
                "enabled": true,
                "server_name": "", // 填入你RealiTLScanner扫描到的域名
                "reality": {
                    "enabled": true,
                    "handshake": {
                        "server": "", // 填入你RealiTLScanner扫描到的域名对应的IP地址
                        "server_port": 443
                    },
                    "private_key": "", // 执行 sing-box generate reality-keypair 生成，注意保存public_key
                    "short_id": [ // 执行 sing-box generate rand 8 --hex 生成
                        ""
                    ]
                }
            }
        }
    ],
    "outbounds": [
        {
            "type": "socks",
            "tag": "proxy-cheap",
            "server": "Your_socks_server_IP", // 替换成你的，下同
            "server_port": "Your_socks_server_Port",
            "version": "5",
            "username": "Your_socks_server_username",
            "password": "Your_socks_server_password"
        },
        {
            "type": "direct",
            "tag": "direct"
        }
    ],
    "route": {
        "rules": [
            {
                "rule_set": [
                    "geosite-anthropic",
                    "geosite-openai"
                ],
                "outbound": "proxy-cheap"
            }
        ],
        "rule_set": [
            {
                "tag": "geosite-anthropic",
                "type": "remote",
                "format": "binary",
                "url": "https://github.com/SagerNet/sing-geosite/raw/rule-set/geosite-anthropic.srs",
                "download_detour": "direct"
            },
            {
                "tag": "geosite-openai",
                "type": "remote",
                "format": "binary",
                "url": "https://github.com/SagerNet/sing-geosite/raw/rule-set/geosite-openai.srs",
                "download_detour": "direct"
            }
        ],
        "final": "direct"
    },
    "experimental": {
        "cache_file": {
            "enabled": true
        }
    }
}
```
写完的json配置可以在[在线编辑器](https://jsonlint.com/)中校对  

`Ctrl+S`保存

启用sing-box

`sudo systemctl enable sing-box`

启动sing-box

`sudo systemctl start sing-box`

查看日志

`sudo journalctl -u sing-box --output cat -e`

重启sing-box

`sudo systemctl restart sing-box`

### v2rayN创建节点  
**enjoy!**  
😊  

> 服务器-->添加VLESS服务器-->复制粘贴节点信息

## 开启BBR  
详见[秋水逸冰](https://teddysun.com/489.html)

# sing-box更新

连接ssh，输入如下命令：
```
bash <(curl -fsSL https://sing-box.app/deb-install.sh)
```
提示类似字样表示更新成功：
```
root@localhost:~# bash <(curl -fsSL https://sing-box.app/deb-install.sh)
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 10.9M  100 10.9M    0     0  3677k      0  0:00:03  0:00:03 --:--:-- 7856k
(Reading database ... 104152 files and directories currently installed.)
Preparing to unpack sing-box.deb ...
Unpacking sing-box (1.9.6) over (1.9.4) ...
Setting up sing-box (1.9.6) ...
```
重启sing-box：`sudo systemctl restart sing-box`
查看日志观察有无报错：`sudo journalctl -u sing-box --output cat -e`