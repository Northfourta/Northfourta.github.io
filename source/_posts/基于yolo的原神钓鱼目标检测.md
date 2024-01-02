---
title: 基于yolo的原神钓鱼目标检测
date: 2023-12-28 18:45:10
tags:

---

## 1、YOLO 环境配置
### 1.1 安装 CUDA 和 cudnn
参考之前写的 [博客](https://blog.csdn.net/weixin_44349241/article/details/114333235)，按其中步骤安装即可。这里安装的是 version 为 11.2 的cuda。
### 1.2 安装 pytorch-gpu
去 [torch官网](https://pytorch.org/get-started/previous-versions/) 寻找与自己 cuda 版本相对应的 pytorch，这里我没有找到对应 cuda-11.2 版本的torch安装命令，于是选择了安装 cuda-11.1 的torch（**自己安装torch的cuda版本应不能超过电脑的 cuda 版本**）
```bash
pip install torch==1.8.1+cu111 torchvision==0.9.1+cu111 torchaudio==0.8.1 -f https://download.pytorch.org/whl/torch_stable.html
```
> 建议先安装 torch，再安装 yolo；若先安装 yolo，系统会自行安装 torch-cpu，还需卸载
### 1.3 安装 yolov8

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
运行
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
```

## 3、训练模型

### 2.1 创建数据加载配置文件

新建data文件夹（可自定义），再在data目录下新建images, labels, data.yaml 

[YOLOv8 从环境搭建到推理训练_yolov8 predict-CSDN博客](https://blog.csdn.net/weixin_61988885/article/details/129421538?share_token=dad53e58-4c3a-4935-9a84-56924d78a2af)

[教程：超详细从零开始yolov5模型训练_yolo训练-CSDN博客](https://blog.csdn.net/qq_45701791/article/details/113992622)

📌images目录下存放数据集的图片文件

📌labels目录下存放txt标准格式标签

📌yaml文件用来存放一些目录信息和标志物分类

---

### 2.2 创建数据集

步骤（YOLO）：
1. 在 data/predefined_classes.txt 文件中定义将用于训练的类别列表。
2. 使用上述说明构建并启动。
3. 在工具栏中的“保存”按钮正下方，点击“PascalVOC”按钮切换到YOLO格式。
4. 您可以使用“打开/Open”或“打开/OpenDIR”来处理单个或多个图像。处理完单个图像后，请点击保存。
   YOLO格式的txt文件将保存在与图像同名的文件夹中。名为“classes.txt”的文件也将保存在该文件夹中。 “classes.txt”定义了YOLO标签所引用的类别列表。

**注意：**

在处理图像列表时，您的标签列表不应该在处理过程中更改。当您保存一张图像时，classes.txt也会被更新，而之前的注释不会被更新。在保存为YOLO格式时，不应使用“默认类别”功能，它不会被引用。保存为YOLO格式时，“difficult”标志会被丢弃。 

### 2.3 开始训练

```bash
yolo task=detect mode=train model=yolov8s.yaml data=mydata_tuomin/tuomin.yaml epochs=100 batch=4 device=0
```

以上参数解释如下：

📌task：选择任务类型，可选['detect', 'segment', 'classify', 'init']。

📌mode: 选择是训练、验证还是预测的任务蕾西 可选['train', 'val', 'predict']。

📌model: 选择yolov8不同的模型配置文件，可选yolov8s.yaml、yolov8m.yaml、yolov8x.yaml等。

📌data: 选择生成的数据集配置文件

📌epochs：指的就是训练过程中整个数据集将被迭代多少次,显卡不行你就调小点。

📌batch：一次看完多少张图片才进行权重更新，梯度下降的mini-batch,显卡不行你就调小点。

---

