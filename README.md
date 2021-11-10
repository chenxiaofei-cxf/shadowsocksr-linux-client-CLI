# shadowsocksr-linux-client-CLI
Linux(`ubuntu 16.10`)上shadowsocksr Python客户端的配置，其实ssr部分在**macos**上也能使用。

## 安装步骤

* 安装shadowsocksR服务端（**在服务端安装**）

> [参考链接](https://shadowsocks.be/9.html)

    # 下载傻瓜式安装脚本`shadowsocksR.sh`，或者文件夹里我们已经为你下载好的安装脚本`shadowsocksR.sh`。
    wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh
    # 修改权限
    chmod +x shadowsocksR.sh
    # 运行脚本并输出日志到文件
    ./shadowsocksR.sh 2>&1 | tee shadowsocksR.log

    # 使用命令
    # 启动
    /etc/init.d/shadowsocks start
    # 停止
    /etc/init.d/shadowsocks stop
    # 重启
    /etc/init.d/shadowsocks restart
    # 状态
    /etc/init.d/shadowsocks status

    # 配置文件路径：/etc/shadowsocks.json
    # 日志文件路径：/var/log/shadowsocks.log
    # 代码安装目录：/usr/local/shadowsocks

* Linux安装shadowsocksR Python客户端（**在本地Linux系统上操作**）

> [参考链接](https://www.djangoz.com/2017/08/16/linux_setup_ssr/)

    wget http://www.djangoz.com/linux_setup_ssr/ssr # ssr脚本已经被下载到git文件夹中
    sudo mv ssr /usr/local/bin
    sudo chmod 766 /usr/local/bin/ssr
    ssr install
    ssr config # 目前只能配置单服务器版本，多服务器版本以后再添加



> shadowsocks以及shadowsocksR服务端本身只能提供socks5代理，但是多数应用使用http/https协议，所以需要一个代理软件把socks5协议的流量转换成http/https的流量，下面是一种小而快的缓存web代理服务器的安装方式。

这边使用privoxy将socks5 代理转换成http代理

* 安装privoxy

```shell
sudo apt-get install proxy
```

* 配置privoxy

sudo vi /etc/privoxy/config 

注释掉 `listen-address `那两行
在最后面添加

```shell
forward-socks5t   /   127.0.0.1:1080 .
listen-address  127.0.0.1:8118
```

* 启动 privoxy

```shell
sudo service privoxy restart
```

* 使用http代理

```shell
export ALL_PROXY='http://127.0.0.1:8118'
```

* 取消代理
```shell
unset ALL_PROXY
```


* 设置SSR服务开机自动启动

> 由于部分常用Linux发行版本将初始化系统从`init.d`改成了`systemd`，且题主正在使用的ubuntu也从16.04开始放弃了`init.d`改为使用`systemd`，所以今天使用systemd配置SSR服务。

    # 创建服务控制文件
    vim /lib/systemd/system/ssr.service

```shell
    # 服务控制文件内容：
    [Unit]
    # 描述服务
    Description=shadowsocksR CLI client
    # 用于指定服务启动的前置条件
    After=network.target
    # 帮助文件的地址如http://baidu.com/ ，可缺省
    Documentation=https://github.com/chenxiaofei-cxf/shadowsocksr-linux-client-CLI
    Wants=network.target

    [Service]
    Type=forking
    # 服务启动命令，此项必填
    ExecStart=/usr/bin/python /usr/local/share/shadowsocksr/shadowsocks/local.py -d start -c /usr/local/share/shadowsocksr/config.json
    # 服务终止命令，可缺省
    ExecStop=/usr/bin/python /usr/local/share/shadowsocksr/shadowsocks/local.py -d stop -c /usr/local/share/shadowsocksr/config.json

    # 用来定义如何启动，以及是否开机启动
    [Install]
    # 当服务开机启动后，会放入什么文件夹，影响启动顺序
    WantedBy=multi-user.target
```

> 之后可使用如下命令来使用ssr服务

    # 修改ssr.service服务控制文件之后刷新守护进程
    sudo systemctl daemon-reload
    # 设置ssr开机启动
    sudo systemctl enable ssr.service
    # 取消ssr开机启动，并取消服务
    sudo systemctl disable ssr.service
    # 开启ssr服务
    sudo systemctl start ssr.service
    # 关闭ssr服务
    sudo systemctl stop ssr.service
    # 查看ssr服务状态
    sudo systemctl status ssr.service
