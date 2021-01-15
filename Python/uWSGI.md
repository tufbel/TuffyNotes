## 1 uWSGI的安装

win系统下无法安装，Linux系统下，直接使用pip命令安装即可。

## 2 uWSGI配置

uWSGI可以直接使用命令行启动，但是由于其配置复杂，建议创建配置文件`[confname].ini`。配置解释如下：

```shell
[uwsgi]
; %v代表pwd的当前路径，并非uwsgi.ini所在路径
; %d代表配置文件uwsgi.ini所在文件夹的绝对路径
chdir = %d/../
; 开放http+socket的端口
;http-socket = :21300
; socket监听队列的小，默认100
listen = 120
socket = myuwsgi.socket
pidfile = uwsgi.pid
; 服务停止时自动清除socket和pid文件
vacuum = true

; 允许主进程存在，并在工作进程中加载flask而非主进程中
master = true
lazy-apps = true
workers = 1
threads = 10
; 启用线程
enable-threads = true
; 序列化接收请求，防止惊群现象发生
thunder-lock = true

; 设置后台运行，并保存日志
daemonize = logs/uwsgi.log
; 禁用请求日志
; disable-logging = true
; 设置日志文件大小
; log-maxsize = 1048576

;可以只是用module，也可以搭配callable使用
;module = your_module_name:app_name
;callable = your_app_name
;virtualenv = your_env_path
; 指定静态文件url和路径，配置nginx不需要单独写了
; static-map=/static=/var/www/orange_web/static
; 放出静态文件可能需要此项
; sgi-disable-file-wrapper = true
```

## 3 uWSGI的启动关闭

各种命令参考如下：

```shell
uwsgi --ini confname.ini  # 通过ini文件启动
uwsgi --reload/--stop uwsgi.pid  # 重载或者停止
uwsgi --help  # 查看帮助
```



