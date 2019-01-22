# Mac80端口问题

## 解除占用

`macOS`的1024以下端口都是不能直接使用的，如果要用得先排出是否已经有进程使用了某个端口：

``` sh
# 查看80端口的占用情况
sudo lsof -i ':80' |grep 'LISTEN'
```

获取到`pid`后`kill`掉。

## 代理转发80端口

实际上在我的机器上默认是没有任何进程在使用80端口的，仍需要`sudo`是因为`macOS`对1024以下端口的保护。

查阅网上很多资料，都说只能通过将所有访问80端口的请求转发到可用端口（例如8080）。

转发的方法有使用`pfctl`的，也有用`nginx`作代理转发的，两种我都试过。考虑到配置的方便(macOS 10以上要处理pfctl没那么容易)，还有对系统的修改，最后决定使用`nginx`做代理解决。

### 安装 nginx

首先你要有个`nginx`，直接`brew`安装就好，没有`brew`那就[戳这里](https://brew.sh/index_zh-cn)。

### 修改nginx配置

默认`brew`安装好后`nginx`的配置文件存放在
```
/usr/local/etc/nginx/nginx.conf
```

默认是监听8080端口的，改成80后将所有请求`proxy_pass`到你想要的端口。

``` sh
 server {
        listen       80;
        server_name  localhost;
        location / {
            proxy_pass   http://127.0.0.1:8080;
            # 下面两句如果有用到ws:// websocket的话就加上
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
}
```
nginx的详细配置方式自行[官网](https://nginx.org/en/docs/)学习，不能面向搜索引擎～

### 启动nginx

因为监听了80端口，启动当然需要`sudo`
``` sh
sudo nginx
```
这样做就能让访问本机80端口的转到8080端口，但，仅限本机访问本机，如果是局域网内其他机器要访问，是会失败的，因为防火墙的存在。

最开始我想着要不关掉防火墙算了，但是存在必定有其道理，想想还是再搜索下折腾下吧，后来找到了一个办法：

``` sh
sudo nginx -g "daemon off;"
```

当然啦，这样只运行实在前台运行的，要走到后台那就`nohup`加`&`处理下。

### 开机启动

这种做法是要自己在终端运行的，如果不想每次重开机就做一次就要自己设置开机启动啦。

最简便的方式就是把这个命令写成以脚本，开机执行就是了。可以参考[这里](http://makaiqian.com/setting-boot/)
