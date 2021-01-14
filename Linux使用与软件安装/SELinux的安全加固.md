存在SELinux的系统，往往限制端口和一些文件的访问

# 1 端口访问规则配置

```shell
firewall-cmd --zone=public --list-ports  # 查看防火墙所有开放的端口

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

netstat -lnpt # 查看进程监听端口
netstat -anp # 查看内容更详细
```

# 2 SELinux配置

## 2.1 查看与修改

SELinux目前一共三种模式：

- enforcing：强制模式，代表 SELinux 运作中，且已经正确的开始限制 domain/type 了；

- permissive：宽容模式：代表 SELinux 运作中，不过仅会有警告讯息并不会实际限制 domain/type 的存取。这种模式可以运来作为 SELinux 的 debug 之用；

- disabled：关闭，SELinux 并没有实际运作。

```shell
getenforce # 查看SELinux当前模式
setenforce [0|1] # 0代表permissive宽容模式，1代表enforcing强制模式
```

使用`sestatus`命令能看到完整的SELinux的运行状态与模式等相关参数：

![SELinux运行状态查看](https://raw.githubusercontent.com/tufbel/TImages/main/mark/Snipaste_2021-01-13_09-44-42.png)

可以通过`status`与`mode`行看到运行状态与当前运行模式。

另外还可以通过修改`/etc/selinux/config`这个SELinux的配置文件来更改它的配置，但是SELinux的配置重载需要计算机重启。

## 2.2 启用和关闭

由于SELinux的启动和关闭需要修改配置文件`/etc/selinux/config`并重启计算机才能生效，所以我们一般通过更改SELinux的模式来达成启用和关闭的效果：

```shell
setenforce 0 # 宽容模式（关闭）
setenforce 1 # 强制模式（启用）

sed -i "s\SELINUX=enforcing\SELINUX=disabled\g"  /etc/selinux/config # 永久关闭，需要重启
```

## 2.3 规则配置

使用semanage命令来为SELinux配置规则，常用参数罗列：

- port: 管理定义的网络端口类型
- fcontext: 管理定义的文件上下文
- -l: 列出所有记录
- -a: 添加记录
- -m: 修改记录
- -d: 删除记录
- -t: 添加的类型
- -p: 指定添加的端口是tcp或udp协议的，port子命令下使用
- -e: 目标路径参考原路径的上下文类型，fcontext子命令下使用

### 2.3.1 端口规则

```shell
semanage port -a -t http_port_t -p tcp 80 # 添加80端口为http_port_t类型，80可更换为80-90则为批量修改80-90端口。
semanage port -l | grep http_port_t # 查看所有端口规则，配合grep使用
semanage port -lC # 查看自己定义的而非SELinux默认的
```

### 2.3.2 文件规则

```shell
ll -Z  # 查看文件被标记的类型，只有被标记为 httpd_sys_content_t 的文件才能被http访问，当然还有其他规则
# 例：以为Nginx 新增一个网站目录 ~/myproject/projectname 为例
semanage fcontext -a -t httpd_sys_content_t "~/myproject/projectname(/.*)?"
semanage fcontext -l # 搭配grep使用
```

