> Python有多种安装方式，但是python开发过程中，需要根据多种应用场景配置多种开发环境，所以python的**虚拟环境**成了必不可少的一环，为了更好的管理python环境，我不推荐使用python官方安装包安装，我更习惯使用**Conda**进行Python的环境管理。

## 1 Conda的安装与使用

### 1.1 下载安装包

从[conda官方](https://docs.conda.io/en/latest/miniconda.html)下载miniconda的安装包，根据自己需要选择版本。

### 1.2 安装

Win系统直接运行`miniconda.exe`安装文件，Linux系统使用`bash`命令。以下以Linux系统为例进行讲解：

```shell
bash Miniconda3-latest-Linux-x86_64.sh # 安装
```

阅读完协议内容后，输入`yes`进行安装。

##### 配置安装位置（建议使用默认位置）

![安装位置配置](https://cdn.jsdelivr.net/gh/tufbel/TImages/mark/20210121102039.png)

默认安装在`~/`文件夹下，如果想自定义位置，则需要指定，例如在`/mnt/d/Linux/Ubuntu1804/dev/miniconda3`。

> 因为miniconda安装默认为用户安装，所以不建议更换安装目录。如果一定要换的话要指定到miniconda3文件夹下，而且如果miniconda3文件夹已存在，会安装失败，但是可以通过参数进行设置。

##### 初始化

安装的最后一步会询问是否执行`conda init`，它会在你的`~/.bashrc`文件中添加conda的环境变量初始化，所以选择yes执行就好。

以上步骤只给当前用户执行了`conda init`，如果切换用户就找不到conda的命令了，所以即使上面选择了no也无所谓，我们可以通过如下方式来为**其他用户**添加**conda命令**。

```shell
su user # 切换到指定用户，也可以sudo su切换到root
cd miniconda3_path/bin # 进入到miniconda安装目录的bin文件夹下
./conda init # 执行conda init命令
```

可以看到`conda init`执行并修改了`~/.bashrc`文件

![执行效果](https://cdn.jsdelivr.net/gh/tufbel/TImages/mark/20210121104138.png)

使用`source ~/.bashrc`命令重新加载一下bash配置，即可使用conda命令了。

### 1.3 配置与换源

**通过命令方式：**

```shell
conda config --set auto_activate_base false  # 设置shell启动不激活base环境
conda config --set show_channel_urls yes # 安装包时显示来源
conda config --set ssl_verify yes # ssl认证
conda config --add channels url # 添加源
conda config --remove-key channels  # 撤销源的设置，恢复默认

# 上海交通大学源
auto_activate_base: false
show_channel_urls: true
channels:
  - https://mirrors.sjtug.sjtu.edu.cn/anaconda/pkgs/main/
  - https://mirrors.sjtug.sjtu.edu.cn/anaconda/pkgs/free/
  - https://mirrors.sjtug.sjtu.edu.cn/anaconda/cloud/conda-forge/
  - defaults
ssl_verify: true
```

**通过修改配置文件：**

Linux用户的配置文件保存在`~/.condarc`中，Win用户的配置文件保存在`C:\User\name\.condarc`文件中，可以通过手动新建，但是建议使用任一命令如：`conda config --set auto_activate_base false`让conda自己来创建它。然后使用vim修改将如下内容添加到其中，可达到与命令方式相同的效果：

```shell
auto_activate_base: false
show_channel_urls: true
channels:
  - https://mirrors.sjtug.sjtu.edu.cn/anaconda/pkgs/main/
  - https://mirrors.sjtug.sjtu.edu.cn/anaconda/pkgs/free/
  - https://mirrors.sjtug.sjtu.edu.cn/anaconda/cloud/conda-forge/
  - defaults
ssl_verify: true
```

### 1.4 Conda使用

##### 环境管理

```shell
conda create -n name [python=3.xxx] [--clone envname] #　创建虚拟环境 --clone用来复制
conda activate name # 激活环境
conda deactivate # 关闭环境
conda remove -n your_env_name --all # 删除

# 虚拟环境导入导出
conda env export --file py36.yaml # 导出为yaml 需要先激活虚拟环境
conda env create -f=py36.yaml # 创建
```

### 1.5 卸载

1. 使用`rm -rf minicondapath`命令删除目录。
2. 编辑`~/.bashrc`或`~/.bash_profile`等文件删除环境变量。
3. 使用`rm -rf ~/.condarc ~/.conda ~/.continuum`删除相关目录。

## 2 Pip

### 2.1 换源

##### 临时使用

```shell
pip install -i [url] arrow #使用-i参数
# 阿里云 https://mirrors.aliyun.com/pypi/simple/
# 清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/
# 中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
```

##### 永久更换

通过命令或者修改配置文件的方式可以使唤pip安装源的永久更换。Linun下文件一般在`~/.config/pip/pip.conf`；win下文件一般在`C:\Users\[Username]\AppData\Roaming\pip\pip.ini`。

```shell
# 永久更换pip源命令
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/

# pip配置文件内容
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host = mirrors.aliyun.com/pypi/simple/
```

### 2.2 环境转移

```shell
pip freeze -r requirements.txt # 导出环境依赖包
pip install -r requirements.txt # 导入环境依赖包
```

