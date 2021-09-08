## 1 安装RabbitMQ

### 1.1 apt安装

**配置签名**

```shell
# 安装apt-transport-https便于下载Rabbit和Erlang的包
sudo apt-get install curl gnupg apt-transport-https -y

## 添加RabbitMQ的signing key
curl -1sLf "https://keys.openpgp.org/vks/v1/by-fingerprint/0A9AF2115F4687BD29803A206B73A36E6026DFCA" | sudo gpg --dearmor > /usr/share/keyrings/com.rabbitmq.team.gpg
## 提供Erlang的Launchpad PPA
curl -1sLf "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0xf77f1eda57ebb1cc" | sudo gpg --dearmor > /usr/share/keyrings/net.launchpad.ppa.rabbitmq.erlang.gpg
## 配置PackageCloud的RabbitMQ repository
curl -1sLf "https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey" | sudo gpg --dearmor > /usr/share/keyrings/io.packagecloud.rabbitmq.gpg
```

**添加Rabbit团队维护的apt源**

```shell
sudo tee /etc/apt/sources.list.d/rabbitmq.list <<EOF
# 然后分别输入Erlang和packagecloud的源 Ubuntu1804为例
deb [signed-by=/usr/share/keyrings/net.launchpad.ppa.rabbitmq.erlang.gpg] http://ppa.launchpad.net/rabbitmq/rabbitmq-erlang/ubuntu bionic main
deb-src [signed-by=/usr/share/keyrings/net.launchpad.ppa.rabbitmq.erlang.gpg] http://ppa.launchpad.net/rabbitmq/rabbitmq-erlang/ubuntu bionic main
deb [signed-by=/usr/share/keyrings/io.packagecloud.rabbitmq.gpg] https://packagecloud.io/rabbitmq/rabbitmq-server/ubuntu/ bionic main
deb-src [signed-by=/usr/share/keyrings/io.packagecloud.rabbitmq.gpg] https://packagecloud.io/rabbitmq/rabbitmq-server/ubuntu/ bionic main
EOF

# 添加完成后更新源
sudo apt-get update
```

以上命令执行完成后，相当于新建了`/etc/apt/sources.list.d/rabbitmq.list`，并添加了上述四条存储库信息。

> 注意上述`bionic`为Ubuntu1804版本，如果是2004应使用`focal`,其他版本也更换成对应名称即可。

**安装**

```shell
## 安装Erlang
sudo apt-get install -y erlang-base \
                        erlang-asn1 erlang-crypto erlang-eldap erlang-ftp erlang-inets \
                        erlang-mnesia erlang-os-mon erlang-parsetools erlang-public-key \
                        erlang-runtime-tools erlang-snmp erlang-ssl \
                        erlang-syntax-tools erlang-tftp erlang-tools erlang-xmerl

## 安装rabbitmq及其依赖
sudo apt-get install rabbitmq-server -y --fix-missing

## 如果使用apt安装下载deb文件太慢或失败，也可以手动下载deb使用dpkg安装
## 但这需要你手动安装rebbitmq所需要的依赖
## 安装依赖
sudo apt-get -y install socat logrotate init-system-helpers adduser
## 下载deb并安装
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.9.5/rabbitmq-server_3.9.5-1_all.deb
sudo dpkg -i rabbitmq-server_3.9.5-1_all.deb
```

## 2 运行RabbitMQ

`RabbitMQ`使用`systemctl`进行管理。

```shell
# 运行成功
sudo service rabbitmq-server start
sudo service rabbitmq-server stop

# 报错
sudo systemctl start rabbitmq-server  # 启动
sudo systemctl stop rabbitmq-server  # 关闭
sudo systemctl enable rabbitmq-server  # 设置开机自启动

sudo rabbitmqctl status  # 查看状态
```

## 3 配置RabbitMQ

#### 3.1 配置文件

`Rabbitmq`默认配置文件为`/etc/rabbitmq/rabbitmq.conf`（需要自己手动创建）。

使用`sudo rabbitmqctl status`可以查看到如下内容：

```shell
...
Config files

 * /etc/rabbitmq/rabbitmq.conf
...
```

## 4 Rabbit GUI

### 4.1 访问UI

RabbitMQ可以使用其自带的`Management Plugin`工具通过浏览器进行管理。其发行版中已经自带了该工具，所以只需要启动即可使用：

```shell
sudo rabbitmq-plugins enable rabbitmq_management
```

然后通过浏览器访问`localhost:15672`即可访问管理页面。

### 4.2 用户管理

使使用Web浏览器管理RabbitMQ需要进行用户验证，如果没有则需要使用`rabbitmqctl`命令进行添加:

```shell
# 创建一个用户
sudo rabbitmqctl add_user [name] [password]
# 标记用户为administrator以赋予用户UI和远程管理权限
sudo rabbitmqctl set_user_tags [name] administrator
```

### 4.3 虚拟机权限

默认新增的用户是没有`Can access virtual hosts`权限，不能访问虚拟机的，可以使用命令修改:

```shell
sudo rabbitmqctl set_permissions [-p <vhostpath>] <user> <conf> <write> <read>
# 例如
set_permissions -p [name] .* .* .*
```

其中`conf`、`write`、`read`位置分别用正则表达式来匹配特定的资源。

或者使用Web页面为用户添加权限：

![选择用户](https://raw.githubusercontent.com/tufbel/TImages/main/mark/20210908094950.png)

依次点击1、2进入`rabbit_`用户的权限管理界面。

![权限管理](https://raw.githubusercontent.com/tufbel/TImages/main/mark/20210908095917.png)

可以看到1处提示没有访问虚拟机的权限，可以在2处选择虚拟机与匹配规则，然后点击`Set Permission`即可。

## 5 运行报错

### 5.1 启动报错

如果使用`systemctl`或者`rabbitmqctl`命令启动`rabbitmq`时报错：

```shell
rabbit@Tuffy-Cowave:
  * connected to epmd (port 4369) on Tuffy-Cowave
  * epmd reports: node 'rabbit' not running at all
                  no other nodes on Tuffy-Cowave
  * suggestion: start the node
```

可以尝试使用`sudo service rabbitmq-server start`来启动`rabbitmq`。

