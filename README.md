# UnLockNeteaseMusic
解锁网易云音乐灰色
#CentOS 7、Debian、Ubuntu
curl -sSL https://get.docker.com/ | sh
systemctl start docker
systemctl enable docker

#参考
#https://www.moerats.com/archives/938/
#https://www.lajiblog.com/index.php/archives/4/

#运行镜像
docker run --restart=always --name unmusic -d -p 7777:8080 nondanee/unblockneteasemusic

#运行完后可通过HTTP proxy 进行解锁
#=========以下为使用shadowsocks 转发==========
#使用glider转发
wget -N --no-check-certificate https://github.com/nadoo/glider/releases/download/v0.7.0/glider-v0.7.0-linux-amd64.tar.gz

tar zxvf glider-v0.7.0-linux-amd64.tar.gz && cd glider-v0.7.0-linux-amd64

vi glider.conf

#输入 i 输入以下内容

#开启调试模式,输出log
verbose=True
#ss的监听端口
listen=ss://CHACHA20-IETF:password@:8888
#网易云音乐解锁代理的端口
forward=http://127.0.0.1:7777
#ss可以改成自己想要的加密方式密码和端口

#切换英文输入法 输入 :wq! 回车保存

#安装screen窗口使其在后台运行
yum install screen 

#创建窗口

screen -S  unlock 

#输入以下内容 ，回车

chmod 777 glider && ./glider -config glider.conf

#按ctrl +a  再按d 让此页面挂在后台
