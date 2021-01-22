## 1 介绍

gcc是Linux系统一个重要的工具，很多软件的安装、源码的编译都需要它，Linux系统默认的gcc版本一般都是**4.8.5**，但是有的时候我们需要用到更高版本的gcc，本文将介绍gcc的版本安装与更换问题。

> 由于Ubuntu系统暂未测试，所以暂以CentOS 7系统为例。

## 2 CentOS

### 2.1 gcc安装与版本管理

为了更好的管理gcc的版本，需要借助`centos-release-scl`。

#### 2.1.1 安装

> 有的系统已经自带了scl，可以使用scl命令检测一次，如果带有scl则直接安装gcc即可。
>
> 注意：有时候会遇到找不到devtoolset-9-gcc的问题，这时候我们可能需要卸载epel，并更换yum源，对所有涉及的包进行重新安装即可。

```shell
yum install centos-release-scl # 安装管理工具
# 安装gcc 版本自行选择 建议使用第一条命令，安装过程中会遇到无公钥的情况，选择y安装即可，或是下面携带-y的参数
yum install [-y] devtoolset-7-gcc*
yum install devtoolset-7-gcc devtoolset-7-gcc-c++ devtoolset-7-binutils 

yum list all |grep -i [name] # 搜索yum可安装的包
```

如果使用`yum repolist`命令可以看到`centos-sclo-rh`与`centos-sclo-sclo`源则可以正常安装`scl`与`gcc`，否则需要进行换源处理。

<img src="https://raw.githubusercontent.com/tufbel/TImages/main/mark/Snipaste_2021-01-12_12-27-10.png" alt="yum repolist列表" style="zoom:67%;" />

#### 2.1.2 管理gcc版本

##### 临时切换

```shell
scl enable devtoolset-7 bash
```

执行上述命令后，在你本次使用的bash中都将使用`devtoolset-7`的gcc，可以通过`gcc -v`命令查看版本是否切换成功。

##### 永久切换（不建议）

```shell
echo “source /opt/rh/devtoolset-7/enable” >> /etc/profile
```

通过添加**全局环境变量**来调用scl的管理命令来更换gcc版本。上述命令作用是在`/etc/profile`文件中添加了一行内容为：`source /opt/rh/devtoolset-7/enable`。

### 2.2 安装epel

epel是一个能为CentOS系统提供高质量的软件包。**无需求可忽略

#### 2.2.1 安装epel

##### 安装

```
yum install epel-release
```

##### 切换其他源的epel

**备份（如果以安装epel请进行备份）**

```shell
mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.backup
mv /etc/yum.repos.d/epel-testing.repo /etc/yum.repos.d/epel-testing.repo.backup
```

**下载最新的epel到`/etc/yum.repos.d/`目录下**：

**1.阿里**

```shell
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo # 从阿里源下载，epel版本请根据自己CentOS系统版本进行选择
```

**2.官方**

有的包阿里源里不具备，所以可以使用官方源：

```shell
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm # 官方源直接安装即可
```

#### 2.2.2 更新并查看

下载好后epel源后使用如下命令更新并查看epel是否已成功添加到yum源：

```shell
yum repolist # 用于查看所有的yum源
```

其他相关命令（更换yum源时经常使用）

```shell
yum clean all # 清除yum缓存
yum makecache # 生成yum缓存
```

#### 2.2.3 卸载epel

如果epel源不全，有时候需要卸载epel重装：

```shell
yum remove epel-release  # 通过yum卸载epel
rm -rf /var/cache/yum/x86_64/6/epel/ # 清空epel目录
```

## 3 Ubuntu

### 3.1 安装

安装`build-essential`，它是一个Ubuntu下的构建开发包整合，里面包括`gcc-7`、`g++-7`、`make`与`manpages-dev`、`update-alternatives`等工具。

```shell
sudo apt install build-essential
gcc --version # 查看gcc版本
```

### 3.2 版本管理

##### 添加最新版本库（如果不需要可以不用）

将`ubuntu-toolchain-r/test PPA`添加到系统，这样才可以使用apt命令安装较新版本的`gcc-9`。

```shell
sudo apt install software-properties-common
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
```

然后安装不同版本的`gcc`即可：

```shell
sudo apt install gcc-7 g++-7 gcc-8 g++-8 gcc-9 g++-9 # 7已经在build-essential中安装的可以忽略
```

##### 管理版本

首先使用`update-alternatives`收录所有安装的版本，数字90、80、90代表优先级，越大代表优先级越高，优先级最高的为默认版本。

```shell
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 90 --slave /usr/bin/g++ g++ /usr/bin/g++-9
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 80 --slave /usr/bin/g++ g++ /usr/bin/g++-8
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 70 --slave /usr/bin/g++ g++ /usr/bin/g++-7
```

然后使用`update-alternatives`的命令就可以更改默认版本了：

```shell
sudo update-alternatives --config gcc
```

