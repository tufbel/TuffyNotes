# 1 安装Nginx

## 1.1 查验是否安装Nginx

通过如下命令检测Linux机器上是否安装了Linux，一般都被安装到`/usr/sbin/nginx`路径下。

```shell
whereis nginx
which nginx
find / -name nginx
```

## 1.2 安装Nginx

### 1.2.1 通过yum或apt安装

直接使用`install`命令安装即可，安装完成后通过`nginx -v`查看版本。

### 1.2.2 安装Nginx官方版本

#### RHEL/CentOS

为了从官方源安装Nginx，首先安装yum的工具包

```shell
sudo yum install yum-utils
```

然后新建`/etc/yum.repos.d/nginx.repo`文件，并在其中写入如下内容：

```shell
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

**默认安装nginx的stable稳定版本，如果想安装mainlin最新版本，还需要运行如下命令：**

```shell
sudo yum-config-manager --enable nginx-mainline
```

之后直接运行`install`安装命令即可。

```shell
sudo yum install nginx
```

安装过程中会校验GPG key，可以通过[Nginx官网](http://nginx.org/en/linux_packages.html#RHEL-CentOS)查看并比对。

#### Ubuntu

同样需要先安装依赖：

```shell
sudo apt install curl gnupg2 ca-certificates lsb-release
```

根据需要安装的版本（**stable或mainlin**）选择如下一个命令运行：

```shell
echo "deb http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
# 安装stable稳定版本
```

```shell
echo "deb http://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

接下来导入官方秘钥：

```shell
curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
```

确认现有秘钥，获得输出内容，输出内容比对同样通过[Nginx官网](http://nginx.org/en/linux_packages.html#Ubuntu)查看：

```shell
sudo apt-key fingerprint ABF5BD827BD9BF62
# 以下内容为输出
pub   rsa2048 2011-08-19 [SC] [expires: 2024-06-14]
      573B FD6B 3D8F BC64 1079  A6AB ABF5 BD82 7BD9 BF62
uid   [ unknown] nginx signing key <signing-key@nginx.com>
```

更新apt源并安装即可：

```shell
sudo apt update
sudo apt install nginx
```

# 2 Nginx启动

## 2.1 启动

Nginx安装完成后，需要手动的启动或者重启

> 在CentOS上使用systemctl，在Ubuntu上使用service。

```shell
systemctl start nginx  # CentOS
systemctl enable nginx # CentOS开机启动Nginx

service nginx start  # Ubuntu

# 通用命令 只能在nginx启动后使用
nginx -s stop/quit/reopen/reload   # 停止/退出/重新打开/重新载入配置
```

然后通过命令查看Nginx的情况以及占用端口：

```shell
ps -ef | grep nginx
# 输出
root  6853  1    0 20:27 ?        00:00:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
nobody 7839  6853  0 21:35 ?        00:00:00 nginx: worker process
```

查看端口使用情况：

```shell
netstat -ntlp|grep nginx # 查看监听端口
# 输出
tcp 0   0   0.0.0.0:80  0.0.0.0:*  LISTEN    97705/nginx: master
```

## 2.2 防火墙检查

执行完**2.1**我们就可以通过`ip:port`的形式访问Nginx，但是此时有的系统因为带有防火墙，所有访问可能会被拦截，这时需要对防火墙进行设置。

### 2.2.1 CentOS

请在如下命令中选择合适命令进行防火墙配置：

```shell
firewall-cmd --zone=public --list-ports  # 查看防火墙所有开放的端口
# 必要三步
firewall-cmd --zone=public --add-port=80/tcp --permanent  # 开放80端口
firewall-cmd --zone=public --add-port=80-90/tcp --permanent # 批量开放80-90端口
firewall-cmd --zone=public --remove-port=80/tcp --permanent  # 关闭80端口
firewall-cmd --zone=public --query-port=80/tcp  # 查看80端口协议是否生效
firewall-cmd --reload # 配置立即生效
firewall-cmd --list-all # 查看防火墙规则

# 不考虑安全性也可以关闭防火墙
systemctl stop firewalld.service # 关闭
systemctl start firewalld.service # 启动
firewall-cmd --state # 查看防火墙状态
netstat -lnpt # 查看监听端口
ps [PID] # 查看进程详情
```

## 2.3 更换端口

### 2.3.1 SELinux限制端口

在具有SELinux的系统上，如果随意更换端口，Nginx的启动会失败，这是因为SELinux限制Nginx只能选择那些被标记为`http_port_t`的端口启动：

<img src="https://raw.githubusercontent.com/tufbel/TImages/main/mark/Snipaste_2021-01-07_12-12-15.png" alt="启动失败" style="zoom:80%;" />

按照提示，使用`journalctl -xe`命令查看错误信息（vim使用 G 移动到底部）：

<img src="https://raw.githubusercontent.com/tufbel/TImages/main/mark/Snipaste_2021-01-07_12-09-36.png" alt="错误提示" style="zoom:67%;" />

其提示让使用`semanage port -a -t PORT_TYPE -p tcp 21300`命令标记21300端口类型。

> SELinux限制只有特定类型的端口才能被某些特定软件访问，semanage是一个用来管理这些端口类型的命令。更多详细讲解查看《SELinux的安全加固》篇。

```shell
semanage port -a -t http_port_t -p tcp 21300  # 设置21300端口类型为http_port_t
semanage port -l | grep http_port_t # 查看http_port_t类型的端口
```

<img src="https://raw.githubusercontent.com/tufbel/TImages/main/mark/20210113170505.png" alt="设置效果" style="zoom:80%;" />

设置完成后效果如图，只后再运行Nginx启动命令即可。

### 2.3.2 修改端口

在Nginx的配置文件的server块中修改port即可，修改完成后，可以使用`netstat -anp |grep nginx`命令查看端口使用情况。

## 2.4 文件访问

SELinux不仅会限制端口，也会限制Nginx对文件的访问：

```shell
setenforce 0  # 临时关闭SELinux(实际设置为宽容模式)
```



# 3 配置Nginx

<img src="https://raw.githubusercontent.com/tufbel/TImages/main/mark/Snipaste_2021-01-07_14-18-45.png" alt="Nginx分层图" style="zoom:67%;" />

Nginx的配置文件一般包括全局块、events块、http块、server块和location块。

1. main全局块：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。
2. events块：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
3. http块：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。
4. server块：配置虚拟主机的相关参数，一个http中可以有多个server。
5. location块：配置请求的路由，以及各种页面的处理情况。



Nginx配置文件包括`/etc/nginx/nginx.conf`与`/etc/nginx/conf.d/*.conf`。

1. nginx.conf文件：main全局块、events块、http块的全局配置一般写在此文件中，并且此文件的http块最后会使用`include /etc/nginx/conf.d/*.conf;`语句导入`conf.d`文件夹中的配置。
2. conf.d文件夹：其中可以包含多个`*.conf`文件，每个conf文件中尽量都只写一个server块，给某一个项目使用，以便使改项目独占一个Nginx虚拟主机。

## 3.1 nginx.conf 基本配置模板

为了提高Nginx的性能，我们可以对Nginx默认配置进行修改和优化。[官方配置讲解](http://nginx.org/en/docs/ngx_core_module.html)

### 3.1.1 全局块配置

```shell
# 系统用户，在启动时因为安全组策略导致不能在只能在固定端口启动就是因为此使用的非root用户
# user user [group];
# 由于有的Linux系统安全加固，导致Nginx不具备访问一些文件的权限，所以可以使用root用户，默认使用nginx用户
user  root; 
# 工作进程数量
worker_processes  1;
worker_rlimit_nofile 65535; # 工作进程打开文件的最大数量，一般为 (ulimit -n)/工作进程数。但是有时候Nginx的分配也不是那么均匀。
worker_cpu_affinity 0001 0010 0100 1000; # 用于给每个工作进程绑定单独的cpu，只设置一个工作进程时候可以不设置此选项

error_log  /var/log/nginx/error.log warn; # 错误日志位置与保存级别
pid        /var/run/nginx.pid; # 主进程pid的存放位置
```

> 注意，`ulimit -n`查看到数值一般为1024，这个是默认值，对Nginx来说一般是不够用的，所以需要手动修改文件句柄数量。

### 3.1.2 events块配置

```shell
events {
		accept_mutex on; #优化同一时刻只有一个请求而避免多个睡眠进程被唤醒的设置(惊群现象：一个请求唤醒多个子进程)，on为防止被同时唤醒，默认为off，因此nginx刚安装完以后要进行适当的优化。
  multi_accept on; # 设置每个进程可以同时接受多个网络连接，默认为off代表不能接受多个网络连接。
  # 事件驱动模型（只能在Linux上使用），select|poll|kqueue|epoll|resig|/dev/poll|eventport
	 use epoll; # 使用epoll事件驱动，因为epoll的性能相比其他事件驱动要好很多，可以不设置，因为Nginx默认使用最有效的。
  # 设置单个工作进程允许的最大连接数量，次数字不能大于当前操作系统支持的最大文件句柄数，可以使用ulimit -n 命令查看操作系统支持的最大句柄数。
  worker_connections 65535; 
}
```

### 3.1.3 http块

```shell
http {
   # 文件扩展名与文件类型映射表。设定mime类型(邮件支持类型),类型由mime.types文件定义
    include     /etc/nginx/mime.types;
   # 默认文件类型，默认为text/plain
    default_type  application/octet-stream;
		# 自定义日志格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarded_for"';
   # 设置访问日志路径和格式。
    access_log  /var/log/nginx/access.log  main;
   # 允许sendfile方式传输文件，可以检测静态文件访问提高服务器性能
    sendfile        on;
    sendfile_max_chunk  100k;  # 设置sendfile最大限制，默认为0，不设限制
    #tcp_nopush     on;
		
		#连接超时时间，默认为75s，可以在http，server，location块。建议针对不同服务单独设置
    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

### 3.1.4 server块

针对不同的项目server块进行不同的配置，以下为示例：

```shell
server {
    listen       21300;
    server_name  localhost;

    sendfile on;

    charset utf-8;
    #最长响应时间
    keepalive_timeout  300;

    #日志路径
    access_log  your_project_path/logs/nginx_access.log;
    error_log  your_project_path/logs/nginx_error.log;

    location /static {
        # alias会把其value与匹配到的路径替换 /static/.. = value/..
        alias /home/tuffy/my-projects/FlaskProject/static;
        # root会把其value路径与匹配到的url拼接  value/static/...
        # root /home/tuffy/my-projects/FlaskProject;
    }

    location / {
        try_files $uri @projectname;
    }

    location @projectname {
        include uwsgi_params;
        # 替换成需要的项目的socket文件所在位置，使用Linux的套接字
        uwsgi_pass unix:uwsgi.sock;
    }
}
```

## 3.2 配置方式

- 推荐为每一个项目配置一个Server块，这样就可以实现在不同端口上启动不同的项目。
- 推荐将配置文件放到项目目录下，然后通过软连接的方式连接到Nginx配置文件夹内。

#### 例

假设项目路径为 `/root/project`，nginx配置文件路径为`/etc/nginx`。通常`nginx.conf`文件会位于`/etc/nginx`目录下其内会有一条关键语句：`include /etc/nginx/conf.d/*.conf;`，代表其将从`/etc/nginx/conf.d`目录中导入所有`*.conf`文件中的配置。

此时我们需要进行三步操作：

1. `vim /root/project/nginx.conf`，编辑配置文件写入Server块内容，并保存。
2. `ln -s /root/project/nginx.conf /etc/nginx/conf.d/projectname.conf`，将配置文件软连接到nginx配置文件目录中。
3. `nginx -s reload`，重新载入nginx配置。

