## 1 安装

一般系统都是默认安装了ssh的，可以通过`whereis ssh`检查，如果未安装，请先安装：

```shell
sudo apt-get install openssh-server # 安装
sudo apt-get remove openssh-server # 卸载
```

> 由于默认的**ssh**没有秘钥，没有秘钥**ssh**是无法建立连接的，所以可以先卸载ssh服务然后重新安装。

## 2 修改配置

### 2.1 修改配置

安装后，通过`vim /etc/ssh/sshd_config`命令来修改**ssh**的配置文件（**记得提前备份**）。

```shell
#取消注释, 修改默认端口号22为25190
Port 25190
#取消注释（可以不用） 限制登录ip
ListenAddress 127.0.0.1
#取消注释,允许root账户登录
PermitRootLogin yes
#开启密码验证,建议开启
PasswordAuthentication yes

# 指定秘钥位置，可以不取消注释使用默认
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key
```

> 修改好配置即可使用`service ssh start`命令启动ssh服务了，此时已经可以使用账号与密码访问ssh了，但是如果连接root用户，会发现无法使用密码连接，这是因为没给root用户设置密码，使用`sudo su`切换到root用户，然后使用`sudo passwd root`设置新密码即可。

### 2.2 配置秘钥访问

#### 客户机创建秘钥

首先使用命令或者其他工具在客户机创建公钥

```shell
ssh-keygen -t rsa -b 4096
```

Win系统会在`C:\Users\username\.ssh`中生成文件夹中，Linux系统生成在``~/.ssh`文件夹中 。`id_rsa`是自己私钥，`id_rsa.pub`是给别人的公钥。

#### 将客户机的秘钥复制到远程终端机

> 将`id_rsa.pub`中的内容复制到`~/.ssh/authorized_keys`文件中即可。如果是`root`用户的话，直接复制粘贴会因为权限问题导致秘钥不能使用，建议使用正确步骤操作。
>

在Git Bash中运行如下命令：

```shell
ssh-copy-id [-i identity_file] -p port user@hostname # ssh-copy-id命令在w10系统下没用的，但是Git bash中有。另外，如果想手动把秘钥复制到authorized_keys文件中，可以使用-i参数，identity_file代表本地秘钥复制到远程机中的文件位置。
```

连接到远程机器，然后可以查看`~/.ssh/authorized_keys`文件的权限为600（root用户与非root用户改文件权限一致）。所以，如果是自己创建`authotized_keys`文件的话也要控制其权限为600，否则ssh是无法使用的，可以使用`chmod`命令：

```shell
chmod 600 ~/.ssh/authorized_keys
```

![authorized_keys权限](https://cdn.jsdelivr.net/gh/tufbel/TImages/mark/20210121091617.png)

## 3 启动

### 3.1 启动方式

```shell
# 如下命令完成ssh的启动与停止
sudo service ssh start
sudo service ssh restart
sudo service ssh stop

# 可以查看ssh对端口的占用情况
netstat -anp|grep ssh
```

### 3.2 配置开机启动

#### 新建 wsl文件

新建`/etc/init.wsl`文件，并在文件中写入启动ssh服务的命令：

```shell
service ssh start
```

#### 添加可执行权限

运行`chmod +x`命令添加执行权限（添加完后使用ls查看会看到文件名字变绿）：

```shell
sudo chmod +x /etc/init.wsl
```

然后编辑`/etc/sudoers`文件，避免输入密码，添加一行：

```shell
%sudo ALL=NOPASSWD: /etc/init.wsl
# 编辑完使用 :wq！退出，记得提前备份
```

#### 添加启动脚本

返回win界面，新建一个txt文件，然后改名为`UbuntuService.bat`脚本（千万不要更改的默认打开方式），添加以下内容：

```shell
# -c表示参数，使用bash运行'sudo /etc/init.wsl'命令
bash -c "sudo /etc/init.wsl"
```

运行bat脚本可以看到能成功子系统的ssh服务，但是会弹出cmd界面，然后瞬间消失，为了避免这种情况，我们使用vbs脚本来启动bat脚本（实测直接用vbs脚本运行bash.exe会失败，可能是机器原因），在`UbuntuService.bat`统计目录下新建`UbuntuService.vbs`脚本，内容为：

```powershell
set ws=WScript.CreateObject("WScript.Shell")
ws.Run "UbuntuService.bat",0
```

按`win+r`，输入`shell:startup`，将所写的vbs脚本文件的快捷方式放入打开的文件夹中即可。

重启测试效果。