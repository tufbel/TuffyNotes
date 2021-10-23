## 前言

WSL2可以在w10中运行Linux虚拟机，并且通过虚拟网卡与宿主机连接，所以默认情况下，宿主机只能使用`127.0.0.1`的地址访问WSL2中的服务。

## 1 配置内网访问

因为WSL2只有宿主机可访问，所以若想内网中其他机器访问WSL2，需要使用宿主机的`端口转发`。

###### 确定wsl虚拟机ip

因为每次启动wsl，其ip都会变更，所以端口转发时应先确定ip：

```shell
# 宿主机
c:> arp -a

# 产生以下输出
接口: 172.29.*.* --- 0x20
  Internet 地址         物理地址              类型
  172.29.*.*            *                   动态

# 在wsl中查看（推荐）
ifconfig
# 输出
eth0: flags=*
      inet 172.29.*.*
```

###### 进行端口映射

使用管理员启动`PowerShell`，`cmd`是没有映射命令的：

```powershell
# 添加端口转发
netsh interface portproxy add v4tov4 listenport=[宿主机windows平台监听端口] listenaddress=0.0.0.0 connectport=[wsl2平台监听端口] connectaddress=[wsl2平台ip]

# 删除端口转发
netsh interface portproxy delete v4tov4 listenport=[宿主机windows平台监听端口] listenaddress=0.0.0.0

# 查看所有端口转发
netsh interface portproxy show all
```

###### 每次开机重新映射问题解决方案

**1.映射回环地址（目前确定Django服务不可用，mysql可用）**

因为使用`127.0.0.1`的回环地址是可以访问到WSL2的虚拟机的，所以可以将端口转发到`127.0.0.1`的回环地址上：

```powershell
# 添加端口转发
netsh interface portproxy add v4tov4 listenport=[宿主机windows平台监听端口] listenaddress=0.0.0.0 connectport=[wsl2平台监听端口] connectaddress=127.0.0.1
```

**2.编写启动脚本(未实现)**

由于每次映射都需要自己手动进行所以就很麻烦，所以可以使用开机启动脚本来自动完成映射。

脚本只需要修改`$ports=@(80);`即可。

```powershell
$remoteport = bash.exe -c "ifconfig eth0 | grep 'inet '"
$found = $remoteport -match '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}';

if( $found ){
  $remoteport = $matches[0];
} else{
  echo "The Script Exited, the ip address of WSL 2 cannot be found";
  exit;
}

#需要修改的部分
$ports=@(80);


#[Static ip]
#You can change the addr to your ip config to listen to a specific address
$addr='0.0.0.0';
$ports_a = $ports -join ",";


#Remove Firewall Exception Rules
#iex "Remove-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' ";

#adding Exception Rules for inbound and outbound Rules
#iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Outbound -LocalPort $ports_a -Action Allow -Protocol TCP";
#iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Inbound -LocalPort $ports_a -Action Allow -Protocol TCP";

for( $i = 0; $i -lt $ports.length; $i++ ){
  $port = $ports[$i];
  iex "netsh interface portproxy delete v4tov4 listenport=$port listenaddress=$addr";
  iex "netsh interface portproxy add v4tov4 listenport=$port listenaddress=$addr connectport=$port connectaddress=$remoteport";
}
```

后续待补充