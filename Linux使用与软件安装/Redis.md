# 1 安装

## 1.1 使用管理工具

通过apt或是yum安装的redis可能版本一般为4.0.9，与其相关gcc版本为4.8.5。若想安装[Stable](https://redis.io/download)版本，请使用1.2的方法官网下载安装包进行安装，同时需要gcc版本大于5.3。

#### CentOS

```shell
yum search redis-server
yum install redis-server
```

#### Ubuntu

```shell
apt search redis-server
apt install redis-server
```

## 1.2 安装包

从[Redis官网](https://redis.io/download)下载安装包，并根据[安装说明](https://redis.io/download#installation)进行安装。

### 1.2.1 下载并解压安装包

```shell
wget https://download.redis.io/releases/redis-6.0.9.tar.gz # 以6.0.9版本为例，可按需更换
tar xzf redis-6.0.9.tar.gz [-C /usr/loacl/]  # 解压 注意：此处解压位置即为redis的安装位置，建议`/usr/loacl/`目录
cd redis-6.0.9 # 进入解压的目录
```

上述下载步骤也可以手动下载压缩包，然后将压缩包上传到Linux系统。

> 后续需要在redis的解压目录中编译，所以解压目录即为redis的安装目录，建议将redis解压目录放到`/usr/local/`文件夹下，解压完成后通过`mv`命令移动即可。

### 1.2.2 编译安装

> 首先是一些可以忽略的特殊操作，按需求选择是否执行即可。

```shell
mv redis-6.0.9 /usr/local/redis-6.0.9 # 移动文件夹
cd /usr/local/ 
ln -s redis-6.0.9 redis  # 进入到/usr/local/目录并未redis-6.0.9文件夹创建软连接，如果无需求可以忽略此步骤，软连接用于管理软件的多版本较为方便，但是redis一般不需要。
```

正式编译安装：

```shell
cd redis-6.0.9
make MALLOC=libc # 编译安装
```

> 注意，make过程需要gcc版本大于5.3，有的Linux系统gcc版本为4.8.5，gcc的版本升级较为复杂，请自行查阅想过步骤。

安装完成后，`redis-server`与`redis-cli`命令都在`redis-6.0.9/src`目录下。

## 1.3 系统级别配置

如果使用的是**安装包方式安装**，为了能在Linux系统中更加方便的使用redis，可以通过一些软连接或者环境变量的方式将`redis-server`和`redis-cli`命令配置到系统级别下。当然如果不需要配置，可以每次进入到redis安装目录执行命令也可以。

### 1.3.1 命令配置

配置环境变量与软连接方式**任选其一**即可。

#### 配置环境变量

使用vim修改`~/.bashrc`文件，添加下面一行：

```shell
export PATH=$PATH:/usr/local/redis/src  # 注意后面路径要根据实际安装位置选择redis-server文件所在路径
# $PATH 代表之前的python，此句作用是在原本的PATH后添加/usr/local/redis/src路径
```

修改完成后重新加载一下bash：

```shell
source ~/.bashrc
```

#### 创建软连接方式

```shell
ln -s /usr/local/redis/src/redis-server /usr/bin/redis-server
ln -s /usr/local/redis/src/redis-cli /usr/bin/redis-cli
```

### 1.3.2 配置文件

通过安装包方式安装的redis的配置文件`redis.conf`默认是放在`$REDISPATH/redis.conf`，即放在redis的make目录下而不是src目录下，而且简单的使用`redis-server`启动服务由于并没有指定配置文件，redis只会使用默认参数启动。

新建`/etc/redis/`目录，并在其中创建`redis.conf`的软连接。

```shell
ln -s /usr/local/redis/redis.conf /etc/redis/redis.conf
```

### 1.3.3 启动与连接

完成上述配置后，就可以启动redis了，但是此时redis默认是前台启动的，所以需要打开两个终端或者配合后台命令使用。

#### 启动

```shell
redis-server /etc/redis/redis.conf # 运行redis
```

然后使用**Ctrl+z**快捷键将redis切换到后台运行，此时任务会暂停，`jobs -l`查看任务的编号，然后通过`bg number`恢复redis的运行，`number`值是通过`jobs`命令搜索到的编号。

<img src="https://raw.githubusercontent.com/tufbel/TImages/main/mark/image-20210112163113375.png" alt="redis运行" style="zoom:67%;" />

#### 连接与关闭

```shell
redis-cli # 运行命令直接连接，更多参数使用 --help查看
# 常用
redis-cli info # 查看redis服务信息
redis-cli shutdown  # 停止redis-server服务
# 如果修改了配置文件，则需要停止redis-server然后重新加载conf文件
# 注意，redis-cli只能通过默认方式连接redis，如果修改了默认端口号，添加了密码等，需要通过-p,-a参数进行指定。
```

# 2 基本配置

## 2.1 重要配置

> 重要配置一定要设置

```shell
requirepass [password] # 访问密码限制，默认被注释，无需密码，bind设置为127.0.0.1时密码是不生效的。
bind 192.168.1.100 10.0.0.1 # 远程访问配置，只允许此处写的ip远程访问，如果为0.0.0.0代表允许任何ip远程访问
port 6379 # 使用的端口号，默认6379 

# 数据文件相关
dbfilename dump.rdb  # 名称，只能是名称
dir /etc/redis # 默认./，这是redis的工作目录，也是redis相关文件的存储位置，一定要修改
daemonize yes # 默认为no，修改为yes使得redis-server后台运行

protected-mode yes # 保护限制，默认yes，如果开启，则必须设置requirepass和bind才允许远程访问，设置为no则直接开启远程访问权限，不建议设置为no。
```

