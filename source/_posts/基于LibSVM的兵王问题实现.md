---

title: 基于LibSVM的兵王问题实现
date: 2022-03-22 23:33:52
categories:
- 学习笔记
tags:
- SVM
- 机器学习
- Python
math: true

---

纸上得来终觉浅，绝知此事要躬行。为了将 LibSVM 库的学习赋予实践，博主在这里基于 Python 的 LibSVM 对浙江大学《机器学习》课程中的兵王问题进行了复现，并以此博客来记录自己的学习过程。 

<!-- more -->

# 一、问题背景

国际象棋的兵王问题：棋盘上黑方只剩一个王，白方剩一个王 一个兵，棋局只有两个结果：“白方将死黑方获胜”或者“和棋”，这个问题属于二分类的问题。这里要利用支持向量机解决这个问题，实现在不告诉计算机国际象棋规则的前提下，让计算机可以根据棋子位置判断棋局的结果。

# 二、数据集

本次用到的数据集为 **UCI Machine Learning Repository:** [Chess (King-Rook vs. King) Data Set](http://archive.ics.uci.edu/ml/datasets/Chess+(King-Rook+vs.+King) )

![](基于LibSVM的兵王问题实现\数据集概况.jpg)

数据集一共包含28056个数据，其中“和棋”样本2796个，“白方胜”样本25260个。数据集的解释如下

![](基于LibSVM的兵王问题实现\兵王问题.jpg)

# 三、LibSVM工具包

LibSVM是由台湾大学林智仁教授等开发设计的SVM工具包，支持C， C++， Java，Python ， R 和 Matlab 等

## 1. 为Python安装LibSVM

利用pip运行如下安装命令：

```python
pip install -U libsvm-official
```

## 2. LibSVM的使用

LibSVM的使用非常简单，只需调用其为我们提供的接口即可，这里我们只需了解其常用的几个接口：

+ svm_problem
+ svm_parameter
+ svm_train
+ svm_predict
+ svm_save_model
+ svm_load_model

具体用法可以参考博主 finley 写的这篇博客：[LibSVM for Python 使用](https://www.cnblogs.com/Finley/p/5329417.html)，个人觉得很详细。

# 四、实现流程

## 1. 预处理数据

首先，尝试加载数据，并观察其内容与分布等基本信息，便于我们后续处理

```python
from libsvm.svmutil import *
import numpy as np
import pandas as pd

# 加载数据
data = pd.read_csv('krkopt.data', header=None)
data.columns=['x1','y1','x2','y2','x3','y3','result'] # 为其columns标签
data.describe() # 观察数据的基本信息
```

![](基于LibSVM的兵王问题实现\数据观察1.jpg)

```python
data.head() # 观察前5个数据
```

![](基于LibSVM的兵王问题实现\数据观察2.jpg)

```python
# 观察数据每列的内容
In[1]:
columns = list(data.columns)
for column in columns:
    print(column, ': ', data[column].unique())
-----------------------------------------------------    
Out[1]:
x1 :  ['a' 'b' 'c' 'd']
y1 :  [1 2 3 4]
x2 :  ['b' 'c' 'd' 'e' 'f' 'g' 'h' 'a']
y2 :  [3 1 2 4 5 6 7 8]
x3 :  ['c' 'd' 'e' 'f' 'g' 'h' 'a' 'b']
y3 :  [2 1 3 4 5 6 7 8]
result :  ['draw' 'zero' 'one' 'two' 'three' 'four' 'five' 'six' 'seven' 'eight'
 'nine' 'ten' 'eleven' 'twelve' 'thirteen' 'fourteen' 'fifteen' 'sixteen']
```

可以发现数据中存在的字符类型，这显然是不行的，我们需要对其进行数值化处理。a，b，c等代表的是棋子的横坐标位置，不妨用1代表a，2代表b，以此类推；同样对于结果而言，利用+1替代draw（正样本），其他则用-1代替（负样本），代码如下：

```python
# 将样本进行数值化
data.replace('a', 1, inplace=True)
data.replace('b', 2, inplace=True)
data.replace('c', 3, inplace=True)
data.replace('d', 4, inplace=True)
data.replace('e', 5, inplace=True)
data.replace('f', 6, inplace=True)
data.replace('g', 7, inplace=True)
data.replace('h', 8, inplace=True)
data.loc[data['result']!='draw', 'result'] = -1
data.loc[data['result']=='draw', 'result'] = 1
```

![](基于LibSVM的兵王问题实现\数值化.jpg)

为了消除指标之间的量纲影响，而对训练造成影响，需要进行数据**标准化处理**：
$$
Xtraining = \frac{Xtraining - meanX}{stdX}
$$

```python
# 标准化
meanX = np.mean(XTraining, axis=0)
stdX = np.std(XTraining, axis=0)
XTraining = (XTraining - meanX) / stdX
XTesting = (XTesting - meanX) / stdX
```

到这里，数据的预处理已经完成，下面我们就可以正式开始构建我们自己的 SVM 模型了。

## 2. 超参数选择

我们选择 RBF 内核的支持向量机，有两个超参数 $C$，$\gamma$ 进行确定。LibSVM 的帮助文档中建议 $C\in [2^{-5}, 2^{15}]$，$\gamma\in[2^{-15},2^{3}]$；这里我们遵循文档规定，在此范围内进行粗略搜索

```python
In[2]:
# 限制超参数的搜索范围
CScale = np.linspace(-5, 15, 11)
GammaScale = np.linspace(-15, 3, 10)
C = np.full_like(CScale, 2) ** CScale
Gamma = np.full_like(GammaScale, 2) ** GammaScale
MAX_ACC = 0
prob  = svm_problem(YTraining, XTraining)
# 进行粗略的超参数的搜索
for i in range(len(C)):
    for j in range(len(Gamma)): 
        # ['-t 2 -c '+ str(C[i]) + ' -g ' + str(Gamma[j]) + ' -v 5']含义：
        # -t 2：			      SVM采用RBF内核
        # -c + str(C[i])：      C 的数值为 C[i]
        # -g + str(Gamma[j])：  gamma 数值为 Gamma[j]
        # -v 5：			      五折交叉验证
        param = svm_parameter('-t 2 -c '+ str(C[i]) + ' -g ' + str(Gamma[j]) + ' -v 5')
        ACC = svm_train(prob, param)
        if ACC > MAX_ACC:
            MAX_ACC = ACC
            index_C = i
            index_Gamma = j
print('最大识别率为：%f\n，C=%f\n,Gamma=%f\n'%(MAX_ACC, C[index_C], Gamma[index_Gamma]))
-----------------------------------------
Out[2]:
Cross Validation Accuracy = 89.98%
Cross Validation Accuracy = 89.98%

.......
Cross Validation Accuracy = 90.32%
Cross Validation Accuracy = 90.24%
Cross Validation Accuracy = 99.28%
Cross Validation Accuracy = 99.3%
Cross Validation Accuracy = 98.96%
最大识别率为：99.420000
C=2048.000000
Gamma=0.031250
```

经过粗略的搜索，发现 $C=2048$，$\gamma=0.3125$ 时，验证集的准确率较高；这就完了？当然没有。也许会有更好结果，说不准呢。我们可以考虑在目前得到的最优值附近进行进一步搜索，期望可以得到意外收获。

```python
In[3]:
# 利用粗略搜索得到的结果限制精细搜索的范围
n = 10
minCScale = (CScale[index_C] + CScale[max(0, index_C-1)]) * 0.5
maxCScale = (CScale[index_C] + CScale[min(len(CScale), index_C+1)]) * 0.5
CScale_finer = np.linspace(minCScale, maxCScale, n+1)

minGammaScale = (GammaScale[index_Gamma] + GammaScale[max(0, index_Gamma-1)]) * 0.5
maxGammaScale = (GammaScale[index_Gamma] + GammaScale[min(len(GammaScale), index_Gamma+1)]) * 0.5
GammaScale_finer = np.linspace(minGammaScale, maxGammaScale, n+1)

C_finer = np.full_like(CScale_finer, 2) ** CScale_finer
Gamma_finer = np.full_like(GammaScale_finer, 2) ** GammaScale_finer
# 进行精细搜索
MaxACC = 0
for i in range(len(C_finer)):
    for j in range(len(Gamma_finer)): 
        param = svm_parameter('-t 2 -c '+ str(C_finer[i]) + ' -g ' + str(Gamma_finer[j]) + ' -v 5')
        ACC = svm_train(prob, param)
        if ACC > MaxACC:    
            MaxACC = ACC
            index_Cfiner = i
            index_Gammafiner = j
print('最大识别率为：%f\nC=%f\nGamma=%f\n'%(MaxACC, C_finer[index_Cfiner], Gamma_finer[index_Gammafiner]))
---------------------------------------------
Out[3]:
Cross Validation Accuracy = 99.34%
Cross Validation Accuracy = 99.36%
Cross Validation Accuracy = 99.32%
.....
Cross Validation Accuracy = 99.24%
Cross Validation Accuracy = 99.24%
Cross Validation Accuracy = 99.06%
Cross Validation Accuracy = 99.24%
Cross Validation Accuracy = 99.3%
Cross Validation Accuracy = 99.2%
最大识别率为：99.540000
C=1176.267116
Gamma=0.041235
```

到这里，我们的超参数的搜索结束了，确定 $C=1176.267$，$\gamma=0.041235$；利用此参数再次训练得到最终的 SVM 模型，并将其存为【model_file】文件，再调用该模型进行测试集的分类，结果显示准确率达到 99.41%

```python
In[4]:
paramTraining = svm_parameter('-t 2 -c '+ str(C_finer[index_Cfiner]) + ' -g ' + str(Gamma_finer[index_Gammafiner]))
model = svm_train(prob, paramTraining)
svm_save_model('model_file', model)
print('test: ')
p_label, p_acc, pval = svm_predict(YTesting, XTesting, model)
----------------------------------------
Out[4]:
test: 
Accuracy = 99.4101% (22920/23056) (classification)
```

# 五、完整代码展示

```python
from libsvm.svmutil import *
import numpy as np
import pandas as pd

# 加载数据
data = pd.read_csv('krkopt.data', header=None)
data.columns=['x1','y1','x2','y2','x3','y3','result'] # 为其columns标签

# 将样本进行数值化
data.replace('a', 1, inplace=True)
data.replace('b', 2, inplace=True)
data.replace('c', 3, inplace=True)
data.replace('d', 4, inplace=True)
data.replace('e', 5, inplace=True)
data.replace('f', 6, inplace=True)
data.replace('g', 7, inplace=True)
data.replace('h', 8, inplace=True)
data.loc[data['result']!='draw', 'result'] = -1
data.loc[data['result']=='draw', 'result'] = 1

# 标准化
meanX = np.mean(XTraining, axis=0)
stdX = np.std(XTraining, axis=0)
XTraining = (XTraining - meanX) / stdX
XTesting = (XTesting - meanX) / stdX

# 限制超参数的搜索范围
CScale = np.linspace(-5, 15, 11)
GammaScale = np.linspace(-15, 3, 10)
C = np.full_like(CScale, 2) ** CScale
Gamma = np.full_like(GammaScale, 2) ** GammaScale
MAX_ACC = 0
prob  = svm_problem(YTraining, XTraining)
# 进行粗略的超参数的搜索
for i in range(len(C)):
    for j in range(len(Gamma)): 
        # ['-t 2 -c '+ str(C[i]) + ' -g ' + str(Gamma[j]) + ' -v 5']含义：
        # -t 2：			      SVM采用RBF内核
        # -c + str(C[i])：      C 的数值为 C[i]
        # -g + str(Gamma[j])：  gamma 数值为 Gamma[j]
        # -v 5：			      五折交叉验证
        param = svm_parameter('-t 2 -c '+ str(C[i]) + ' -g ' + str(Gamma[j]) + ' -v 5')
        ACC = svm_train(prob, param)
        if ACC > MAX_ACC:
            MAX_ACC = ACC
            index_C = i
            index_Gamma = j
print('---------------------------\n粗略搜索完成！')
print('粗略搜索下的结果：\n最大识别率为 %f\n，C=%f\n,Gamma=%f\n'%(MAX_ACC, C[index_C], Gamma[index_Gamma]))

# 利用粗略搜索得到的结果限制精细搜索的范围
n = 10
minCScale = (CScale[index_C] + CScale[max(0, index_C-1)]) * 0.5
maxCScale = (CScale[index_C] + CScale[min(len(CScale), index_C+1)]) * 0.5
CScale_finer = np.linspace(minCScale, maxCScale, n+1)

minGammaScale = (GammaScale[index_Gamma] + GammaScale[max(0, index_Gamma-1)]) * 0.5
maxGammaScale = (GammaScale[index_Gamma] + GammaScale[min(len(GammaScale), index_Gamma+1)]) * 0.5
GammaScale_finer = np.linspace(minGammaScale, maxGammaScale, n+1)

C_finer = np.full_like(CScale_finer, 2) ** CScale_finer
Gamma_finer = np.full_like(GammaScale_finer, 2) ** GammaScale_finer
# 进行精细搜索
MaxACC = 0
for i in range(len(C_finer)):
    for j in range(len(Gamma_finer)): 
        param = svm_parameter('-t 2 -c '+ str(C_finer[i]) + ' -g ' + str(Gamma_finer[j]) + ' -v 5')
        ACC = svm_train(prob, param)
        if ACC > MaxACC:    
            MaxACC = ACC
            index_Cfiner = i
            index_Gammafiner = j
print('---------------------------\n精细搜索完成！')
print('精细搜索下的结果：\n最大识别率为 %f\nC=%f\nGamma=%f\n'%(MaxACC, C_finer[index_Cfiner], Gamma_finer[index_Gammafiner]))

# 构建 SVM 模型
paramTraining = svm_parameter('-t 2 -c '+ str(C_finer[index_Cfiner]) + ' -g ' + str(Gamma_finer[index_Gammafiner]))
model = svm_train(prob, paramTraining)
svm_save_model('model_file', model)
print('test: ')
p_label, p_acc, pval = svm_predict(YTesting, XTesting, model)
```

