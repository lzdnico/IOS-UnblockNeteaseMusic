# UnLockNeteaseMusic

通过nginx代理实现ios 免自签证书解锁网易云

详细参考链接<br>
https://www.moerats.com/archives/938/<br>
https://www.lajiblog.com/index.php/archives/4/ <br>
https://github.com/nondanee/UnblockNeteaseMusic/issues/65
<br>
为了简洁以下以 CentOS 7、Debian、Ubuntu 为参照系统
<br>如需详细配置，查阅以上链接<br>

`打开vps所用端口，自行谷歌`<br>`如果不理解，请搜索 如何打开xx云所有端口`
<br>准备域名解析到自己的VPS上，并且申请ssl证书，腾讯云免费证书：https://console.cloud.tencent.com/ssl <br>
1.安装 nginx 
```
yum install nginx

cd /etc/nginx

mkdir cert
```
申请的证书复制到cert文件里，可以用Xftp上传<br>
配置nginx
```
vim nginx.conf
```
将server 配置改成如下：
```
    server {
        listen       443;
        #listen       [::]:80 default_server;
        server_name  您的域名;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
        ssl on;
        ssl_certificate cert/xxx.crt;# 改为自己申请得到的 crt 文件的名称
        ssl_certificate_key cert/xxx.key;# 改为自己申请得到的 key 文件的名称
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;
        location / {
                proxy_pass http://localhost:8080;#转发到unblock代理
        }
    }

```
切换英文输入法 按` Esc `然后输入`:wq!` 回车保存
启动 `nginx` 并设置为开机启动
```
sudo systemctl start nginx.service

sudo systemctl enable nginx.service
```
在浏览器输入`https://你的域名`能打开nginx 页面为成功，因为设置了转发可能会显示 `error`

2. 安装nodejs
```
#CentOS系统
curl -sL https://rpm.nodesource.com/setup_10.x | bash -

yum install nodejs git -y
```
2. 安装screen窗口
```
yum install screen

screen -S node  #新建一个窗口
```
3. 运行UnblockNeteaseMusic
```
git clone https://github.com/nondanee/UnblockNeteaseMusic.git

cd UnblockNeteaseMusic

node app.js -e https://你的域名   #可以通过 -p <port> 来更换端口和nginx转发端口保持一致
```
按ctrl +a  再按d 让此页面挂在后台


## 运行完后可通过HTTP PROXY进行解锁
http-proxy: `你的域名:8080`
<br>可直接使用在Window 客户端(建议http测试成功再进行ss转发)
<br>如果不需要ss转发，只是用http 代理，无需继续下面的教程
<br>
## 以下为使用shadowsocks 转发
#### 需要完成以上http的搭建，不可跳过直接进行ss转发

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
forward=http://127.0.0.1:8080
#ss可以改成自己想要的加密方式密码和端口
```
切换英文输入法 按` Esc `然后输入`:wq!` 回车保存

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
以 Quantumult X 配置为例
<br>

```
server_local
shadowsocks=your_ip:8888,method=chach20-ietf,password=password,fast-open=false,udp-relay=false,tag=Unblock

policy
static=NeteaseMusic,Unblock,direct

Rule
host-suffix, music.163.com, NeteaseMusic
host-suffix, music.126.net, NeteaseMusic
user-agent, NeteaseMusic*, NeteaseMusic
user-agent, NeteaseMusic**, NeteaseMusic
user-agent, 网易云音乐*, NeteaseMusic
user-agent, 网易云音乐**, NeteaseMusic
user-agent, %E7%BD%91%E6%98%93%E4%BA%91%E9%9F%B3%E4%B9%90*, NeteaseMusic
user-agent, %E7%BD%91%E6%98%93%E4%BA%91%E9%9F%B3%E4%B9%90**, NeteaseMusic
ip-cidr, 59.111.181.0/24, NeteaseMusic
ip-cidr, 115.236.118.0/24, NeteaseMusic
ip-cidr, 223.252.199.0/24, NeteaseMusic
host-keyword, netease, NeteaseMusic
```
