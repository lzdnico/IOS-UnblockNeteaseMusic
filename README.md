# UnLockNeteaseMusic

Quantumult X简单方式使用Shadowsocks解锁网易云音乐灰色资源

详细参考链接
https://www.moerats.com/archives/938/

https://www.lajiblog.com/index.php/archives/4/
<br>
为了简洁以下以 CentOS 7、Debian、Ubuntu 为参照系统
<br>如有详细配置，查阅以上链接
```
curl -sSL https://get.docker.com/ | sh

systemctl start docker

systemctl enable docker
```
运行镜像

```
docker run --restart=always --name unmusic -d -p 7777:8080 nondanee/unblockneteasemusic
```
## 运行完后可通过HTTP PROXY进行解锁
http-proxy: `your_ip:7777`
<br>
## 以下为使用shadowsocks 转发

使用glider转发
```
wget -N --no-check-certificate https://github.com/nadoo/glider/releases/download/v0.7.0/glider-v0.7.0-linux-amd64.tar.gz

tar zxvf glider-v0.7.0-linux-amd64.tar.gz && cd glider-v0.7.0-linux-amd64

vi glider.conf
```
按 `i` 输入以下内容
```
开启调试模式,输出log
verbose=True
#ss的监听端口
listen=ss://CHACHA20-IETF:password@:8888
#网易云音乐解锁代理的端口
forward=http://127.0.0.1:7777
#ss可以改成自己想要的加密方式密码和端口
```
切换英文输入法 输入` Esc `然后输入`:wq!` 回车保存

安装screen窗口使其在后台运行
```
yum install screen 
```
创建窗口
```
screen -S  unlock 
```
输入以下内容 ，回车
```
chmod 777 glider && ./glider -config glider.conf
```
按ctrl +a  再按d 让此页面挂在后台

在代理工具中配置Shadowsocks，以`glider.conf` 为准
<br>ss://CHACHA20-IETF:password@:8888
 
## 完成后 http-proxy & shadowsocks 皆可以使用
<br>
以 Quantumult X 为例
<br>

```
server_local
shadowsocks=your_ip:8888,method=chach20-ietf,password=password,fast-open=false,udp-relay=false,tag=Unblock

policy
static=NeteaseMusic,Unblock,direct

Rule
user-agent, NeteaseMusic**, NeteaseMusic
user-agent, NeteaseMusic*, NeteaseMusic
host-suffix, music.163.com, NeteaseMusic
host-suffix, music.126.net, NeteaseMusic
ip-cidr, 59.111.181.0/24, NeteaseMusic
ip-cidr, 115.236.118.0/24, NeteaseMusic
ip-cidr, 223.252.199.0/24, NeteaseMusic
host-keyword, netease, NeteaseMusic
```
