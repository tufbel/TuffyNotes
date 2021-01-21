> 选用PicGo与Github仓库搭建笔记图片仓库

## 1 下载安装PicGo

首先从PicGo的[github仓库](https://github.com/Molunerfinn/PicGo)中下载并安装PicGo。

<img src="https://raw.githubusercontent.com/tufbel/TImages/main/mark/Snipaste_2021-01-11_16-43-14.png" style="zoom:50%;" />

> 上图中有个链接格式，一般推荐选择URL格式，它的作用是在图片上传完成后，将图片的url按照你选择的格式复制到剪贴板。

PicGo还有很多扩展插件，可在[GitHub仓库](https://github.com/PicGo/Awesome-PicGo)中查看。

## 2 创建GitHub的公开仓库

### 2.1 创建仓库

在GitHub中创建一个公开仓库。

### 2.2 创建Token

在Github账户的用户个人设置中`Settings/Developer settings/Personal access tokens`点击右侧`Generate new token`按钮创建一个Token。

<img src="https://raw.githubusercontent.com/tufbel/TImages/main/mark/20210111170221.png" alt="创建Token" style="zoom:80%;" />

给改Token添加说明，方便以后使用，而且其作为一个图片仓库的Token，不用分配过多权限，下面只勾选`repo`即可：

> 注意：创建完成后会出现一个Token，但是它只在创建时显示一次，所以一定要记住，提前复制。

<img src="https://raw.githubusercontent.com/tufbel/TImages/main/mark/20210111170529.png" alt="创建token" style="zoom: 50%;" />

## 3 配置PicGo

### 3.1 PicGo上传设置

在PicGo的图床设置中找到GitHub图床，并将其设置为默认图床：

<img src="https://raw.githubusercontent.com/tufbel/TImages/main/mark/20210111171218.png" alt="github图床设置" style="zoom: 80%;" />

点击确认完成设置。

- 设置仓库名：即为你创建的图片仓库的左上角显示名称，一般为`user/repo`格式。（未试过分组仓库）

- 设置分支名：git仓库分支名称，一定要写对。

- Token：即为步骤2中创建的Token，如果丢了可以重新获取。

- 指定存储路径：其作用是在仓库中新建一个文件夹来保存图片，格式为：`name/`，后面的 `/` 一定要带，否则是在图片名称前拼接改name，依然存储在顶级目录上。

- 设置自定义域名：通过自定义域名，来借用`jsdelivr`这个免费开源的CDN进行图片的加速，格式如下：

  ```shell
  https://cdn.jsdelivr.net/gh/user/repo # CDN加速
  https://raw.githubusercontent.com/user/repo/branch # GitHub原始域名
  
  eg:
  https://raw.githubusercontent.com/tufbel/TImages/main
  https://cdn.jsdelivr.net/gh/tufbel/TImages
  ```

### 3.2 剪贴板上传

PicGo有个快捷上传，通过按下快捷键能直接将剪贴板中的图片上传到图片仓库，**快捷键修改**可以在**PicGO设置**中找到，建议修改为一个不常用的，我的是`Ctrl+ALt+P`。

## 4 配置Typora

打开Typora的偏好设置在图像中进行设置（[官方说明](https://support.typora.io/Upload-Image/#picgoapp-chinese-language-only)）：

<img src="https://raw.githubusercontent.com/tufbel/TImages/main/mark/20210111172703.png" alt="Typora设置" style="zoom:80%;" />

选择`上传图片`，勾选`对本地位置图片应用上述规则`，然后在上传服务中配置PicGo即可（要配置PicGo(app)而不是PicGo-Core）。

