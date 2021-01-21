> Pycharm为开发者提供了方便的**远程开发功能**，包括文件修改的**自动同步**、**远程使用解释器**等功能。下面将介绍其使用方法。

# 1 SSH远程服务器配置

首先打开**Settings --> Tools --> SSH Configurations**进行远程服务器ssh配置，此配置过程要求服务器打开OpenSSH服务。

<img src="https://cdn.jsdelivr.net/gh/tufbel/TImages/mark/20210114152258.png" alt="ssh配置" style="zoom:80%;" />

其中**Visible only for this project** 仅对**本项目使用选项**请根据实际需求选择是否勾选。

配置完成后点击**Test Connection**测试能否连接成功，若成功连接，则点击**Apply**保存配置即可。

# 2 远程解释器

## 2.1 添加远程解释器

在 **Settings --> Project --> Python Interpreter** 设置中配置远程解释器，点击**add**出现解释器配置界面。

1. 选择**SSH Interpreter**使用远程解释器。
2. 选择**Existing server configuration**导入ssh服务器，然后选择刚才配置好的ssh，点击**next**。
3. 选择服务器上的**python解释器路径**，是否使用sudo权限运行代码根据自己需求决定是否勾选。
4. 配置**Running code on  the remote server**，将本地项目文件同步到服务器上，并在服务器上运行代码，下面的**自动同步**功能尽量勾选，配置完成后点击**finish**等待Pycharm配置完成。

<img src="https://cdn.jsdelivr.net/gh/tufbel/TImages/mark/20210114152442.png" alt="配置图1" style="zoom: 80%;" />

<img src="https://cdn.jsdelivr.net/gh/tufbel/TImages/mark/20210114152535.png" alt="配置图2" style="zoom:80%;" />

点击**Finish**后，Pycharm会提示以远程项目以创建，然后我们可以在**Settings --> Deployment**中看到我们刚才配置的项目。

![配置图](https://cdn.jsdelivr.net/gh/tufbel/TImages/mark/20210114103517.png)

## 2.2 远程终端

点击**Tools --> Start SSH Session**然后选择对应服务器即可开启远程终端。

# 3 文件同步

我们默认设置的是文件自动同步，但是有的时候文件修改后并没有自动同步，所以我们可以通过手动Upload的方式同步文件。

## 3.1 手动同步

选择**右键需要同步的文件/文件夹 --> Deployment --> Upload to **即可。

<img src="https://cdn.jsdelivr.net/gh/tufbel/TImages/mark/20210114105624.png" alt="设置图" style="zoom: 67%;" />

然后我们就可以在下方工具栏里的**File Transfer**中看到刚才的上传情况。

## 3.2 查看远程文件

在上方工具栏依次点击**Tools --> Deployment --> Browse Remote Host**，然后选择对应服务器即可查看。

<img src="https://cdn.jsdelivr.net/gh/tufbel/TImages/mark/20210114105936.png" alt="查看远程文件图1" style="zoom:67%;" />

当前文件同步的远程文件背景颜色会变成绿色。

<img src="https://cdn.jsdelivr.net/gh/tufbel/TImages/mark/20210114110458.png" alt="查看远程文件图2" style="zoom:67%;" />

有时候，我们需要Pycharm自己创建一些文件夹用于同步和文件落地，可以打开创建空目录功能。操作入口是**Tools --> Deployment --> Options**选择下图的**Create empty directories**。

