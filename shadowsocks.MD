### 安装shadowsocks流程

环境：centos7.3  

#### 1. 安装pip（Centos7下一般默认自带pip）  
```
yum -y install epel-release
yum -y install pip python-pip
```
对安装好的pip进行升级
```
pip install –upgrade pip
```
#### 2. 安装shadowsocks
```
pip install shadowsocks
```
#### 3. 修改配置  
位置：/etc/shadowsocks.json  
```
{
  "server": "0.0.0.0",
  "local_address": "127.0.0.1",
  "port_password": {
    "8388": "123456"
  },
  "timeout": 300,
  "method": "aes-256-cfb",
  "fast_open": true
}
```
#### 4. 添加开机启动  
位置：/usr/lib/systemd/system/ss.service  
```
[Unit]
Description=Shadowsocks Server
After=network.target

[Service]
PermissionsStartOnly=true
ExecStartPre=/bin/mkdir -p /run/shadowsocks
ExecStartPre=/bin/chown nobody:nobody /run/shadowsocks
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json
Restart=on-abort
User=nobody
Group=nobody
UMask=0027

[Install]
WantedBy=multi-user.target

```
开启开机启动
```
systemctl enable ss
```
启动
```
systemctl start ss
```

#### 5. 完毕!

最近在测试阿里云香港服务器，感觉比vultr快很多。  
用过的：  
[vultr](https://www.vultr.com)，2.5美元、3.5美元、5美元等。  
[阿里云轻服务器](https://www.aliyun.com/product/swas)，最低有24元/月。  
