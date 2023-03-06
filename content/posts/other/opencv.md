---
title: "Opencv"
date: 2021-08-04T10:18:31+08:00
author: ["loveyu"]
draft: false
categories: 
- 其他
tags: 
- openCV
---



### 创建窗口：

>#### *cv2.namedWindow(c1, c2)*
>
>c1：窗口名称
>
>c2：窗口类型；默认为：cv2.WINDOW_AUTOSIZE（不可拉伸）;cv2.WINDOW_NORMAL（窗口大小可拉伸）



### 设置窗口大小：

>#### *cv2.resizeWindow(c1, c2, c3)*
>
>c1：窗口名称
>
>c2：宽
>
>c3：高



### 加载图片：

>#### *img = cv2.imread(c1, c2)*
>
>c1：图片位置
>
>c2：读取图像颜色类型；一般为cv2.IMREAD_COLOR



### 显示图片：

>#### *cv2.imshow(c1, c2)*
>
>c1：窗口名称
>
>c2：cv2.imread返回值或读取视频的帧



### 等待：

>#### *key = cv2.waitKey(c1)*
>
>c1：等待时间；相当于sleep睡眠
>
>key：返回值为键盘监听返回值



### 销毁窗口：

>#### *cv2.destroyAllWindows()*
>
>销毁全部窗口；销毁指定的为去掉all的，参数为指定窗口



### 载入视频：

>#### *video = cv2.VideoCapture(c1)*
>
>c1：当参数为0(数字)则为调用本地摄像头；当参数为视频路径则为加载视频
>
>video.isOpened()：摄像头是否打开；打开返回true否则返回false



### 读取视频：

>#### *ret, frame = video.read()*
>
>video：载入视频的返回值
>
>ret：是否读取成功
>
>frame：读取帧



### 释放视频窗口资源：

>#### *video.release()*
>
>video：载入视频方法的返回值



### 保存视频：

>#### *fourcc = cv2.VideoWriter_fourcc(*"X264")*
>
>参数为：保存视频的格式
>
>vw = cv2.VideoWriter("t.mp4", fourcc, 25, (1280, 720))
>
>参数为：视频保存位置，格式，帧数，大小



### 监听鼠标回调：

>#### *cv2.setMouseCallback(c1, c2, c3)*
>
>c1：监听的窗口名称
>
>c2：回调函数
>
>c3：传入回调函数的参数数据
>
>回调函数：
>
>```python
>def mouse_callback(event, x, y, flags, userdata):
>print(event, x, y, flags, userdata)
>```
>
>userdata：即c3参数值



### 创建Trackbar：

>#### *cv2.createTrackbar(c1, c2, c3, c4, c5)*
>
>c1：trackbar名称
>
>c2：窗口名称，用于添加到指定的窗口
>
>c3：默认值
>
>c4：最大值
>
>c5：回调函数



### 获取track bar数据：

>#### *v = cv2.getTrackbarPos(c1, c2)*
>
>c1：track bar名称
>
>c2：窗口名称
>
>v：返回值为当前track bar的值
>
>案例：通过获取track bar的值设置图片背景颜色
>
>```python
>import cv2
>import numpy as np
>
>
>def callback():
>pass
>
>
>cv2.namedWindow("trackbar")
>
>cv2.createTrackbar("R", "trackbar", 0, 255, callback)
>cv2.createTrackbar("G", "trackbar", 0, 255, callback)
>cv2.createTrackbar("B", "trackbar", 0, 255, callback)
>
>img = np.zeros((480, 460, 3), np.uint8)
>
>while True:
>rv = cv2.getTrackbarPos("R", "trackbar")
>gv = cv2.getTrackbarPos("G", "trackbar")
>bv = cv2.getTrackbarPos("B", "trackbar")
># 给图片的所有像素修改颜色
>img[:] = [bv, gv, rv]
>cv2.imshow("trackbar", img)
>key = cv2.waitKey(10)
>if key * 113:
>   break
>
>cv2.destroyAllWindows()
>```



### 颜色空间转换：

>#### *cvt_img = cv2.cvtColor(c1, c2)*
>
>c1：为图片
>
>c2：cv2.COLOR_BGR2RGB, cv2.COLOR_BGR2BGRA, cv2.COLOR_BGR2GRAY等等，不同参数的颜色不同
>
>例子：
>
>```python
>import cv2
>
>
>def callback():
>pass
>
>
>cv2.namedWindow("color", cv2.WINDOW_NORMAL)
>img = cv2.imread("1.png")
>colorSpaces = [cv2.COLOR_BGR2RGB, cv2.COLOR_BGR2BGRA, cv2.COLOR_BGR2GRAY]
>cv2.createTrackbar("curcolor", "color", 0, len(colorSpaces) - 1, callback)
>
>while True:
>index = cv2.getTrackbarPos("curcolor", "color")
># 颜色空间转换
>cvt_img = cv2.cvtColor(img, colorSpaces[index])
>cv2.imshow("color", cvt_img)
>k = cv2.waitKey(1)
>if k * 113:
>   break
>
>cv2.destroyAllWindows()
>```



### numpy创建矩阵

>#### *b = np.array([1, 2, 3], [4, 5, 6])*
>
>这里可以是任何数组
>
>
>
>创建全是0的矩阵 (行个数，宽个数，层数)，数据类型，因为bgr是3通道的所以这里设置为3，uint8最大值为255
>
>#### *c = np.zeros((8, 8, 3), np.uint8)*
>
>ones全是1的矩阵
>
>#### *d = np.ones((8, 8, 3), np.uint8)*
>
>创建full矩阵，255位自定制值
>
>#### *e = np.full((8, 8, 3), 255, np.uint8)*
>
>定义单位矩阵，对角线值为一样
>
>#### *f = np.identity(4)*
>
>例如：
>
>[[1. 0. 0. 0.]
>[0. 1. 0. 0.]
>[0. 0. 1. 0.]
>[0. 0. 0. 1.]]
>
>#### *g = np.eye(5)*
>
>eye三个参数，如果只有一个参数例如5则为：（5，5，0）
>
>c1：行数；c2：列数；c3：数值起始索引
>
>[[1. 0. 0. 0. 0.]
>[0. 1. 0. 0. 0.]
>[0. 0. 1. 0. 0.]
>[0. 0. 0. 1. 0.]
>[0. 0. 0. 0. 1.]]
>
>#### *g = np.eye(5, 7)*
>
>[[1. 0. 0. 0. 0. 0. 0.]
>[0. 1. 0. 0. 0. 0. 0.]
>[0. 0. 1. 0. 0. 0. 0.]
>[0. 0. 0. 1. 0. 0. 0.]
>[0. 0. 0. 0. 1. 0. 0.]]
>
>#### *g = np.eye(5, 7, 1)*
>
>[[0. 1. 0. 0. 0. 0. 0.]
>[0. 0. 1. 0. 0. 0. 0.]
>[0. 0. 0. 1. 0. 0. 0.]
>[0. 0. 0. 0. 1. 0. 0.]
>[0. 0. 0. 0. 0. 1. 0.]]



### numpy检索复制

>```python
>import cv2
>import numpy as np
>
>img = np.zeros((480, 640, 3), np.uint8)
>
># 只设置一个像素点太小看不出来，设置一竖列更明显
># 这里是y，x不是x，y
>for i in range(100):
># BGR三层，0是第一层是B（blue）
>img[i, 100, 0] = 255	#这样相当于是img[i, 100] = [255, 0, 0]
># 也可以：img[i, 100] = [0, 0, 255]，这样是对[i,100]的第一层赋值0第二层赋值0第三层赋值255是红色；可以根据rgb三色素来设置各种颜色
>
>cv2.imshow("t", img)
>
>key = cv2.waitKey(0)
>if key * 113:
>cv2.destroyAllWindows()
>```



### 获取子矩阵

>```python
>img = cv2.imread("1.png") # 加载图片
>
>roi = img[100:400, 100:600] # 获取img的一部分矩阵；[y1:y2,x1:x2]；获取全部则为[:,:]
>
>roi[200:300, 200:300] = [0, 0, 255] # 设置这一部分矩阵为红色
>```





### 图像属性：

>```python
>img = cv2.imread("1.png")
># img.shape 图片参数； 返回值：(1436, 800, 3)高度，长度，通道数
>print(img.shape)
># img.size 图片大小; 返回值：3446400 图片占用空间 = 高度 * 长度 * 通道数
>print(img.size)
># img.dtype 图片位深；返回值：uint8
>print(img.dtype)
>```



### 通道分离与合并

>#### *b, g, r = cv2.split(img)*
>
>bgr分别为img的三层；bgr分别为一个1层的图像
>
>#### *img2 = cv2.merge((b, g, r))*
>
>img2为bgr三层合并到一起一个新图像



### 绘制

>*画线*
>
>#### *cv2.line(img, (10, 20), (300, 400), (0, 0, 255), 5, 16)*
>
>参数：图像，起始位置，终止位置，颜色，线宽，锯齿度（-1，4，8，16）数值越大越平滑，默认8；还有第七个参数为shift坐标缩放比例（不使用）
>
>*画椭圆*
>
>cv2.ellipse(img, (320, 240), (100, 50), 0, 0, 360, (0, 0, 255), -1)
>
>参数：图像，中心点，长宽(一半)，旋转角度（顺时针），起始角度，最终角度，颜色，填充
>
>*画圆*
>
>cv2.circle(img, (320, 240), 100, (0, 0, 255), -1)
>
>参数：图像，中心点，半径，颜色，填充
>
>*画矩形*
>
>cv2.rectangle(img, (10, 10), (100, 100), (0, 0, 255),  -1)
>
>参数：图像，起始位置，对角线终点位置，颜色，填充
>
>*画多边形*
>
>pts = numpy.array([(300, 10), (150, 100), (450, 100)], np.int32)
>
>参数：点的位置，三个点就是三角形，四个点就是四边形以此类推，最后必须是int32
>
>cv2.polylines(img, [pts], True, (0, 0, 255))
>
>参数：图像，点集，是否闭合，颜色
>
>*多边形填充*
>
>cv2.fillPoly(img, [pts], (0, 255, 0))
>
>参数：图像，点集，颜色
>
>*画文本*
>
>cv2.putText(img, "hello world!", (10, 100), cv2.FONT_HERSHEY_PLAIN, 4, (0, 255, 0))
>
>参数：图像，文本，位置，字体，字号，颜色



### 图像加减乘除

>```python
># 加
>img3 = cv2.add(img, img2)
># 减
>img4 = cv2.subtract(img, img2)
># 乘
>cv2.multiply(img, img2)
># 除
>cv2.divide(img, img2)
>```
>
>减/除 为 A减/除B
>
>*熔合*
>
>img5 = cv2.addWeighted(A,a,B,b,w)
>
>参数：A图像1，a 大A图像的权重，B图像2，b大B图像权重，w静态权重
>
>



### 非与或

>```python
># 非
>img2 = cv2.bitwise_not(img)
># 与
>img2 = cv2.bitwise_and(img, img3)
># 或
>img2 = cv2.bitwise_or(img, img3)
># 异或
>img4 = cv2.bitwise_xor(img, img3)
>```
>





### 图像缩放

>new = *cv2.resize(img, None, fx=0.3, fy=0.3, interpolation=cv2.INTER_AREA)*
>
>new = *cv2.resize(img, (int(h / 3), int(w / 3)))*
>
>resize(c1, c2, c3, c4, c5)
>
>c1：图像；c2：缩放后图像的宽高；c3：x轴缩放因子；c4：y轴缩放因子；c5：缩放算法
>
>算法分别为：
>
>INTER_NEAREST：临近插值，速度快，效果差
>
>INTER_LINEAR：双线性插值，原图中的4个点
>
>INTER_CUBIC：三次插值，原图中的16个点
>
>INTER_AREA：效果最后



### 图像翻转

>#### *cv2.flip(img, c1)*
>
>img：图像
>
>c1：为0上下翻转；>0左右；<0上下+左右



### 图像旋转

>#### *cv2.rotate(img, c2)*
>
>c2：为旋转角度，分别为：cv2.ROTATE_90_COUNTERCLOCKWISE；ROTATE_180；ROTATE_90_COUNTERCLOCKWISE：顺时针90；180；逆时针90





### 仿射变换

>img = cv2.imread("1.png")
>h, w, c = img.shape
>m = *numpy.float32([[1, 0, 100], [0, 1, -200]])*
>new = *cv2.warpAffine(img, m, (w, h))*
>
>这里m必须是float32，100为x轴平移量，-200位y轴平移量，正数向右/向下，负数向左/向上











