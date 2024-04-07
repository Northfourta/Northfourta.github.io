---
title: 基于yolo的原神钓鱼目标检测
date: 2023-12-28 18:45:10
categories:
- 项目
tags:
- YOLO
- 机器学习
- Pytorch
---

原神启动！原批变身！恰好把之前抽的雷神培养一下，正好没有专武，想着把鱼叉弄一下。自己又不想钓鱼，正好就试着把原神自动钓鱼项目[（基于深度强化学习的原神自动钓鱼AI）](https://github.com/IrisRainbowNeko/genshin_auto_fish)复现一下 ，于是便有了这篇博文。

<!-- more -->

## 1、YOLO 环境配置
### 1.1 安装 CUDA 和 cudnn
参考之前写的 [博客](https://blog.csdn.net/weixin_44349241/article/details/114333235)，按其中步骤安装即可。这里安装的是 version 为 11.2 的cuda。

### 1.2 安装 pytorch-gpu

去 [torch官网](https://pytorch.org/get-started/previous-versions/) 寻找与自己 cuda 版本相对应的 pytorch，这里我没有找到对应 cuda-11.2 版本的torch安装命令，于是选择了安装 cuda-11.1 的torch（**自己安装torch的cuda版本应不能超过电脑的 cuda 版本**）
```bash
pip install torch==1.8.1+cu111 torchvision==0.9.1+cu111 torchaudio==0.8.1 -f https://download.pytorch.org/whl/torch_stable.html
```


> 建议先安装 torch，再安装 yolo；若先安装 yolo，系统会自行安装 torch-cpu，还需卸载

### 1.3 安装 YOLOv8

git 拉取 [ultralytics: YOLOv8 🚀 Ultralytics 同步更新官方最新版 YOLOv8 (gitee.com)](https://gitee.com/monkeycc/ultralytics):

```bash
git clone https://gitee.com/monkeycc/ultralytics.git
```

进入 `ultralytics` 目录内，安装 yolo 及相关依赖

```bash
pip install -e .
```
验证是否安装成功
```python
import ultralytics
ultralytics.checks()
```
```bash
Ultralytics YOLOv8.0.230 � Python-3.8.8 torch-1.8.1+cu111 CUDA:0 (NVIDIA GeForce GTX 1050 Ti, 4096MiB)
Setup complete ✅ (4 CPUs, 15.9 GB RAM, 13.4/50.0 GB disk)
```

### 1.4 下载预权重

下载训练模型，推荐yolov8s.pt或者yolov8n.pt，模型小，下载快，在gitee或者github下方readme里面，下载完成后，将模型放在主文件夹下

yolov8s.pt下载地址：[yolov8s.pt](https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8s.pt)

yolov8n.pt下载地址：[yolov8n.pt](https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8n.pt)

### 1.5 测试环境

```bash
yolo predict model=yolov8n.pt source='ultralytics/assets/bus.jpg'
```

``` image 1/1 E:\myProject\ultralytics\ultralytics\assets\bus.jpg: 640x480 4 persons, 1 bus, 1 stop sign, 80.1ms
Speed: 0.0ms preprocess, 80.1ms inference, 3.0ms postprocess per image at shape (1, 3, 640, 480)
Results saved to runs\detect\predict
💡 Learn more at https://docs.ultralytics.com/modes/predict
```

## 2、安装数据标注工具

### 2.1 标注工具基本介绍

这里只介绍两种标注工具：`labelimg`、`labelme`；LabelMe和LabelImg都是用于图像标注的工具，但它们有一些不同之处：

1. **开发者和平台：**
   - LabelMe：LabelMe是由麻省理工学院（MIT）开发的在线标注工具，允许用户标注图像，并支持多种标注格式。
   - LabelImg：LabelImg是由Tzutalin开发的开源工具，它是一个基于Python的图像标注工具，可以在本地使用，支持多种格式的图像标注。
3. **功能：**
   - LabelMe：LabelMe提供了一些高级的标注功能，如<u>实例分割（不仅限于矩形框标注）和复杂形状标注</u>的能力。
   - LabelImg：LabelImg相对简单，适用于一般的对象检测任务，支持<u>常见的矩形标注和类别标签</u>。
4. **使用场景和灵活性：**
   - LabelMe：<u>适合需要更复杂标注需求（如实例分割）的项目</u>，同时需要在线协作或访问的团队。
   - LabelImg：<u>适合简单的对象检测标注需求</u>，更适合个人或小团队在本地使用。

选择使用哪个工具取决于你的具体需求和标注的复杂程度。如果你需要更高级的标注功能并且团队需要在线协作，LabelMe可能更适合；而如果你只需要简单的对象检测标注，LabelImg可能更加方便。这里我们只是进行简单的目标检测，选择使用LabelImg；当然你也可以选用LabelMe，注意LabelMe标注生成的文件为json，还需额外脚本将其生成txt才能用作yolo训练。

### 2.2 LabelMe安装

参考 [GitHub主页](https://github.com/labelmeai/labelme)安装：
```bash
conda create --name=labelme python=3
conda activate labelme
pip install labelme

# or install standalone executable/app from:
# https://github.com/wkentaro/labelme/releases
```
运行命令，启动 labelimg
```bash
labelme
```


### 2.3 LabelImg安装

参考 [github主页](https://github.com/HumanSignal/labelImg)安装步骤安装：

```bash
conda install pyqt=5
conda install -c anaconda lxml
pyrcc5 -o libs/resources.py resources.qrc
python labelImg.py
python labelImg.py [IMAGE_PATH] [PRE-DEFINED CLASS FILE]
----------------------------
pip install labelimg -i https://pypi.tuna.tsinghua.edu.cn/simple
```

## 3、训练模型

### 3.1 创建数据加载配置文件

新建data文件夹（可自定义），再在data目录下新建images, labels, data.yaml 

[YOLOv8 从环境搭建到推理训练_yolov8 predict-CSDN博客](https://blog.csdn.net/weixin_61988885/article/details/129421538?share_token=dad53e58-4c3a-4935-9a84-56924d78a2af)

[教程：超详细从零开始yolov5模型训练_yolo训练-CSDN博客](https://blog.csdn.net/qq_45701791/article/details/113992622)

📌images目录下存放数据集的图片文件

📌labels目录下存放txt标准格式标签

📌yaml文件用来存放一些目录信息和标志物分类

---

### 3.2 创建数据集

步骤（YOLO）：

1. 在 data/predefined_classes.txt 文件中定义将用于训练的类别列表。
2. 使用上述说明构建并启动。
3. 在工具栏中的“保存”按钮正下方，点击“PascalVOC”按钮切换到YOLO格式。
4. 您可以使用“打开/Open”或“打开/OpenDIR”来处理单个或多个图像。处理完单个图像后，请点击保存。
   YOLO格式的txt文件将保存在与图像同名的文件夹中。名为“classes.txt”的文件也将保存在该文件夹中。 “classes.txt”定义了YOLO标签所引用的类别列表。

**注意：**

在处理图像列表时，您的标签列表不应该在处理过程中更改。当您保存一张图像时，classes.txt也会被更新，而之前的注释不会被更新。在保存为YOLO格式时，不应使用“默认类别”功能，它不会被引用。保存为YOLO格式时，“difficult”标志会被丢弃。

### 3.3 数据集划分

 在前面创建的imges及labels文件夹下存放划分后的数据集

```python
import os, shutil, random
from tqdm import tqdm

"""
标注文件是yolo格式（txt文件）
训练集：验证集：测试集 （7：2：1） 
"""


def split_img(img_path, label_path, split_list):
    try:
        Data = './genish_auto_fish'
        # Data是你要将要创建的文件夹路径（路径一定是相对于你当前的这个脚本而言的）
        # os.mkdir(Data)

        train_img_dir = Data + '/images/train'
        val_img_dir = Data + '/images/val'
        test_img_dir = Data + '/images/test'

        train_label_dir = Data + '/labels/train'
        val_label_dir = Data + '/labels/val'
        test_label_dir = Data + '/labels/test'

        # 创建文件夹
        os.makedirs(train_img_dir)
        os.makedirs(train_label_dir)
        os.makedirs(val_img_dir)
        os.makedirs(val_label_dir)
        os.makedirs(test_img_dir)
        os.makedirs(test_label_dir)

    except:
        print('文件目录已存在')

    train, val, test = split_list
    all_img = os.listdir(img_path)
    all_img_path = [os.path.join(img_path, img) for img in all_img]
    # all_label = os.listdir(label_path)
    # all_label_path = [os.path.join(label_path, label) for label in all_label]
    train_img = random.sample(all_img_path, int(train * len(all_img_path)))
    train_img_copy = [os.path.join(train_img_dir, img.split('\\')[-1]) for img in train_img]
    train_label = [toLabelPath(img, label_path) for img in train_img]
    train_label_copy = [os.path.join(train_label_dir, label.split('\\')[-1]) for label in train_label]
    for i in tqdm(range(len(train_img)), desc='train ', ncols=80, unit='img'):
        _copy(train_img[i], train_img_dir)
        _copy(train_label[i], train_label_dir)
        all_img_path.remove(train_img[i])
    val_img = random.sample(all_img_path, int(val / (val + test) * len(all_img_path)))
    val_label = [toLabelPath(img, label_path) for img in val_img]
    for i in tqdm(range(len(val_img)), desc='val ', ncols=80, unit='img'):
        _copy(val_img[i], val_img_dir)
        _copy(val_label[i], val_label_dir)
        all_img_path.remove(val_img[i])
    test_img = all_img_path
    test_label = [toLabelPath(img, label_path) for img in test_img]
    for i in tqdm(range(len(test_img)), desc='test ', ncols=80, unit='img'):
        _copy(test_img[i], test_img_dir)
        _copy(test_label[i], test_label_dir)


def _copy(from_path, to_path):
    shutil.copy(from_path, to_path)


def toLabelPath(img_path, label_path):
    img = img_path.split('\\')[-1]
    label = img.split('.')[0] + '.txt'
    return os.path.join(label_path, label)


if __name__ == '__main__':
    img_path = './genish_auto_fish/imagesAll'  # 你的图片存放的路径（路径一定是相对于你当前的这个脚本文件而言的）
    label_path = './genish_auto_fish/labelsAll'  # 你的txt文件存放的路径（路径一定是相对于你当前的这个脚本文件而言的）
    split_list = [0.7, 0.2, 0.1]  # 数据集划分比例[train:val:test]
    split_img(img_path, label_path, split_list)
```

脚本运行后，生成将标注好的数据随机划分到各自的文件夹中，label同理

![](image-20240103110930577.png)

在对应yaml配置文件中配置好自己数据集的相关信息，以备训练

![](image-20240103111433341.png)

### 3.4 模型训练

```bash
yolo task=detect mode=train model=yolov8s.yaml data=mydata_tuomin/tuomin.yaml epochs=100 batch=4 device=0
```

以上参数解释如下：

📌task：选择任务类型，可选['detect', 'segment', 'classify', 'init']。

📌mode: 选择是训练、验证还是预测的任务类型，可选['train', 'val', 'predict']。

📌model: 选择yolov8不同的模型配置文件，可选yolov8s.yaml、yolov8m.yaml、yolov8x.yaml等，也可选择已经训练好的预训练权重（yolov8s.pt、yolov8m.pt）。

​	选择.pt和.yaml的区别（[YOLOv8训练参数详解](https://blog.csdn.net/qq_37553692/article/details/130898732)）

+ .pt类型的文件是从预训练模型的基础上进行训练。若我们选择 yolov8n.pt这种.pt类型的文件，其实里面是包含了模型的结构和训练好的参数的，也就是说拿来就可以用，就已经具备了检测目标的能力了，yolov8n.pt能检测coco中的80个类别。假设你要检测不同种类的狗，那么yolov8n.pt原本可以检测狗的能力对你训练应该是有帮助的，你只需要在此基础上提升其对不同狗的鉴别能力即可。但如果你需要检测的类别不在其中，例如口罩检测，那么就帮助不大。
+ `.yaml`文件是从零开始训练。采用`yolov8n.yaml`这种.yaml文件的形式，在文件中指定类别，以及一些别的参数。

📌data: 选择生成的数据集配置文件，即前面的fish.yaml

📌epochs：训练的轮次数量，指的就是训练过程中整个数据集将被迭代多少次。

📌batch：每批图像数量（-1为自动批大小）；一次看完多少张图片才进行权重更新，梯度下降的mini-batch，显卡不行你就调小点。

📌device：可以使用`device`参数指定训练设备。如果没有传递参数，并且有可用的GPU，则将使用GPU `device=0`，否则将使用`device=cpu`。

> 更详细的介绍，可前往查阅 yolov8技术文档的训练章节（ [训练 - Ultralytics YOLOv8 文档](https://docs.ultralytics.com/zh/modes/train/#_4)）

训练完成后，系统会输出权重储存路径：

![](image-20240103212134460.png)

## 4、 对原神窗口进行实时目标检测

网上查询了窗口图像采集的方案，最终选定pywin32库进行程序窗口的图像采集

```python
import win32gui, win32ui, win32con
import cv2
import numpy
from ultralytics import YOLO

if __name__ == '__main__':
    window_name = "原神"
    # 获得窗口句柄
    hWnd = win32gui.FindWindow(None, window_name)
    # 加载训练好的yolo模型
    model = YOLO('best.pt')
    # 获取句柄窗口的大小信息
    left, top, right, bot = win32gui.GetClientRect(hWnd)
    print(left, top, right, bot)
    width = (right - left)
    height = (bot - top)
    # 命名输出的窗口名
    cv2.namedWindow('im_opencv')
    # 返回句柄窗口的设备环境，覆盖整个窗口，包括非客户区，标题栏，菜单，边框
    hWndDC = win32gui.GetWindowDC(hWnd)
    # 创建设备描述表
    mfcDC = win32ui.CreateDCFromHandle(hWndDC)
    # 创建内存设备描述表
    saveDC = mfcDC.CreateCompatibleDC()
    # 创建位图对象准备保存图片
    saveBitMap = win32ui.CreateBitmap()

    while True:
        # 为bitmap开辟存储空间
        saveBitMap.CreateCompatibleBitmap(mfcDC, width, height)
        # 将截图保存到saveBitMap中
        saveDC.SelectObject(saveBitMap)
        # 保存bitmap到内存设备描述表
        saveDC.BitBlt((0, 0), (width, height), mfcDC, (0, 0), win32con.SRCCOPY)
        # opencv+numpy保存
        ##获取位图信息
        signedIntsArray = saveBitMap.GetBitmapBits(True)
        ## 图像转换成ndarray
        im_opencv = numpy.frombuffer(signedIntsArray, dtype='uint8')
        im_opencv.shape = (height, width, 4)
        im_opencv = cv2.cvtColor(im_opencv, cv2.COLOR_BGRA2BGR)
        im_opencv = cv2.resize(im_opencv, (0, 0), fx=0.5, fy=0.5)
        # cv2.imwrite("im_opencv.jpg", im_opencv, [int(cv2.IMWRITE_JPEG_QUALITY), 100])  # 保存
        # 利用yolo模型进行目标检测
        result = model(im_opencv)
        cv2.imshow("im_opencv", result[0].plot())  # 显示
        key = cv2.waitKey(10)
        if key & 0xFF == ord('q'): # 识别到按'q'键，退出
            print('正在退出窗口')
            cv2.destroyAllWindows()
            break
```

下面展示一下最终效果，由于本人精力有限，只采集了150张图片作为数据集，整体识别效果一般；但是通过这么一个实现过程，确实能够帮助大家入门yolo

![](原神目标识别.gif)

Reference：

1. [Ultralytics YOLOv8 文档](https://docs.ultralytics.com/zh/quickstart/#conda-docker)
2. [win10下Tensorflow与Pytorch安装教程-CSDN博客](https://blog.csdn.net/weixin_44349241/article/details/114333235)
3. [PyTorch文档](https://pytorch.org/get-started/previous-versions/)
4. [labelImg README zh](https://github.com/HumanSignal/labelImg/blob/master/readme/README.zh.rst)
5. [YOLOv8 从环境搭建到推理训练_yolov8 predict-CSDN博客](https://blog.csdn.net/weixin_61988885/article/details/129421538?share_token=dad53e58-4c3a-4935-9a84-56924d78a2af)
6. [教程：超详细从零开始yolov5模型训练_yolo训练-CSDN博客](https://blog.csdn.net/qq_45701791/article/details/113992622)
7. [YOLOv8训练参数详解（全面详细、重点突出、大白话阐述小白也能看懂）_yolov8参数-CSDN博客](https://blog.csdn.net/qq_37553692/article/details/130898732)
