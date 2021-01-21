## 1 文件读写基本操作

### 1.1 图片

1. `cv2.imread(path,flags) -> img` 读取图像

   - path：图片路径
   - flags： 图片读取模式
     - cv2.IMREAD_COLOR:  BGR格式读取图片
     - cv2.IMREAD_GRAYSCALE:  读取灰度图像
   - img： 读取到的图片，numpy格式

   > 读取带汉字的图片

   ```python
   img = cv2.imdecode(np.fromfile(ProjecPath.get_path('测试.jpg'), dtype=np.uint8), -1)
   ```

2. `cv2.imwrite(name, img)` 保存图像

   - name：保存图片的名字
   - img:：BGR或者灰度的图像数据

   > 保存带汉字的图片

   ```python
    cv2.imencode('.jpg', img)[1].tofile(imgpath)
   ```

3. 枚举参数

   - flags

     | **枚举名**       | **数值** | **含义**   |
     | :--------------- | :------- | :--------- |
     | IMREAD_UNCHANGED | -1       | 原格式读入 |
     | IMREAD_GRAYSCALE | 0        | 灰度图像   |
     | IMREAD_COLOR     | 1        | RGB        |

### 1.2 视频和摄像头

1. `cv2.VideoCapture(path | 0 | num) -> cap ` 读取视频或者获取摄像头

   - path:：视频路径，为读取视频
   - 0 or num：0为使用默认摄像头，num则为其他摄像头索引
   - cap：一个视频或者摄像头对象

   > 使用完成后一定要使用`release()`方法释放资源

   ```python
   def read_video(video_path=None):
       vc = cv2.VideoCapture(video_path)
   			# vc.open() 若为摄像头，可使用此方法打开摄像头
       while vc.isOpened(): # 检验是否成功
           ret, frame = vc.read() # 读出一帧
           yield frame
   
       vc.release() # 释放资源
   ```

2. `cv2.VideoWrite(path, fourcc, fps, frameSize, [isColor]) -> out` 保存视频

   - path：保存路径，包括后缀名
   - fourcc：指定编码器
   - fps：每秒帧率
   - frameSize：视频尺寸（width，height）
   - isColor：指明彩色还是黑白
   - out：视频对象

   `cv2.VideoWriter_fourcc(list) -> fourcc` 初始化编码器

   - list：字符列表，因为此方法不接受字符串，` *'XVID'` 与 ` 'X','V','I','D' ` 效果相同
   - fourcc：编码器对象

   > list可选参数：
   >
   > Fedora中（一个Linux发行版）：DIVX , XVID , MJPG , X264 , WMV1 , WMV2等
   >
   > Windows中：DIVX（.avi） mp4v（.mp4）
   >
   > OSX中：MJPG，DIVX，X264（.mkv）

   ```python
   cap = cv2.VideoCapture(0) # 获取摄像头
   
   # 初始化解码器和视频对象
   fourcc = cv2.VideoWriter_fourcc(*'XVID') 
   out = cv2.VideoWriter('testwrite.avi',fourcc, 20.0, (1920,1080),True)
    
   while cap.isOpened():
       ret, frame = cap.read()
       if ret:
           out.write(frame)
       else:
           break
    
   cap.release()
   out.release()
   ```

### 1.3 从内存中读取

1. `imdecode(buf,flags) -> mat` 从内存缓冲区获取图像
   - buf：数组或者字节向量，python中一般为numpy数组
   - flags：图片读取格式
   - mat：图片对象
2. `imencode(ext,img) ->ret,buf` 将图像编码到缓冲区
   - ext：编码格式 '.jpg'、'.png'等
   - flags：图片格式，RGB或灰度等
   - buf：得到的缓冲区数据，numpy格式
   - ret：编码结果，是否成功

## 2 图片的基本操作

### 2.1 颜色空间

1. `cv2.cvtColor(img, code) -> new_img` 更改图像颜色格式
   - img：图片数据
   - code：转码方向，例如` cv2.COLOR_BGR2RGB`是BGR格式转RGB
   - new_img：调整后图片，此操作不改变原图

### 2.2 图片显示

1. `cv2.imshow(name, img)` 显示图片

   `cv2.namedWindow(name, flags)` 初始化窗口

   `cv2.waitKey(num)` 阻塞程序，窗口等待

   `cv2.destoryAllWindows()` 销毁所有窗口

   - name：窗口名字
   - img：图片数据
   - flags：窗口大小设置，0表示窗口大小可调
   - num：等待时间，单位ms，若设为0，则表示一直等待，直到按下任意键。另一种用法，`cv2.waitKey(10) & 0xFF == ord('q')` 等待10ms或者键盘按下q键

   ```python
   img = cv2.imread('test,png')
   cv2.namedWindow('Image',0)
   cv2.imshow('Image',img)
   if cv2.waitKey(1000) & 0xFF == ord('q'): # 是零x非 大写Ox
     print('显示完成')
   cv2.destoryAllWindows()
   ```

2. 枚举参数

   - flags

     | 名字                | 值         | 作用             |
     | ------------------- | ---------- | ---------------- |
     | WINDOW_NORMAL       | 0x00000000 | 可随意调整大小   |
     | WINDOW_AUTOSIZE     | 0x00000001 | 按图像大小显示   |
     | WINDOW_FULLSCREEN   | 1          | 全屏             |
     | WINDOW_KEEPRATIO    | 0x00000000 | 按照图像比例     |
     | WINDOW_GUI_EXPANDED | 0x00000000 | 显示状态和工具栏 |

### 2.3 绘制图形

1. `line(img,pt1,pt2,color,thickness) -> img` 绘制线段

   `arrowedLine(img,pt1,pt2,color,thickness) -> img` 绘制带箭头的线段

   - img：原始图像会被改变
   - pt1：起始点，(x, y)
   - pt2：终点
   - color：线段颜色，（B,G,R）
   - thickness：线段宽度，单位px

2. `circle(img,center,radius,color,thickness) -> img` 绘制圆

   - center：圆心，(x, y)
   - radius：半径

3. `drawMarker(img,point,color,markerType,markerSize,thickness) -> img` 绘制记号

   - markerType：记号类型

4. `rectangle(img,p1,p2,color,thickness) -> img` 绘制矩形

   - p1：左上角，(x, y)
   - p2：右下角

5. `putText(img, text, org, fontFace, fontScale, color, thckness) -> img` 添加文字

   - org：文本左下角位置
   - fontFace：字体类型
   - fontScale：字体缩放

6. 枚举参数

   - `markerType`

     | 参数                    | 作用          |
     | ----------------------- | ------------- |
     | MARKER_STAR             | 米字型        |
     | MARKER_TILTED_CROSS     | 倾斜45°的加号 |
     | MARKER_DIAMOND          | 菱形          |
     | MARKER_SQUARE           | 正方形        |
     | MARKER_TRIANGLE_UP/DOWN | 上下三角形    |

   - `fontFace`

     | 参数                        | 作用                   |
     | --------------------------- | ---------------------- |
     | FONT_HERSHEY_PLAIN          | 小号加粗               |
     | FONT_HERSHEY_DUPLEX         | 加粗                   |
     | FONT_HERSHEY_COMPLEX        | 一种字体类型，正常粗细 |
     | FONT_HERSHEY_TRIPLEX        | 加粗                   |
     | FONT_HERSHEY_SCRIPT_COMPLEX | 斜字体类型，加粗       |

### 2.4 图像缩放

1. `resize(img, dsize) -> img` 调整图片大小，不改变原图
   - dsize：需要的图片大小
2. `copymakeBorder(img,top, bottom, left, right, borderType，value)` 添加边框，不改变原图大小
   - top，bottom，left，right：向四边扩充的大小
   - borderType：扩充的模式
   - value：若使用纯色扩充，此为RGB值

| 参数                 | 作用             |
| -------------------- | ---------------- |
| cv2.BORDER_REPLICATE | 使用边界像素扩充 |
| cv2.BORDER_REFLECT   | 边界对称扩充     |
| cv2.BORDER_CONSTANT  | 纯色扩充         |

