---
title: 用 scikit-learn 解决兵王问题
date: 2022-04-06 13:45:21
categories:
- 学习笔记
tags:
- scikit-learn
- SVM
- 机器学习
- Python
math: true
---

最近在看机器学习的经典教材《Hands On Machine Learning with Scikit-Learn、Keras and Tensorflow》，其中大量篇幅涉及到 scikit-learn 库。就上手学了学，为了避免自己成为行动上的矮子，想着把它用起来。恰好前几天刚刚用 LibSVM 复现了兵王问题，不如这次就再用 scikit-learn 再复现一次。好吧，其实就是我懒。唉，尽学卡普空炒冷饭了！

<!-- more -->

首先可以确定兵王问题是一个分类问题。显然，我们要采用监督式学习，那就依旧采用 SVM 算法吧。既然是炒冷饭，那就炒到底！！！确定了基本框架之后，就可以开始写代码了。思路与上一次基本相同，除使用库不同外，另外一个区别就是在测试集的生成方面采用了分层抽样的思想。

# 一、加载数据

老样子，先加载数据看看里子是啥样，知己知彼才能事半功倍


```python
import pandas as pd
import numpy as np
data = pd.read_csv('krkopt.data', names=['x1','y1','x2','y2','x3','y3','result'])
data.head() # 查看数据集的前五条
```

**Out [1]:** 

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: center;">
      <th></th>
      <th>x1</th>
      <th>y1</th>
      <th>x2</th>
      <th>y2</th>
      <th>x3</th>
      <th>y3</th>
      <th>result</th>
    </tr>
  </thead>
  <tbody>
    <tr style="text-align: center;">
      <th>0</th>
      <td>a</td>
      <td>1</td>
      <td>b</td>
      <td>3</td>
      <td>c</td>
      <td>2</td>
      <td>draw</td>
    </tr>
    <tr style="text-align: center;">
      <th>1</th>
      <td>a</td>
      <td>1</td>
      <td>c</td>
      <td>1</td>
      <td>c</td>
      <td>2</td>
      <td>draw</td>
    </tr>
    <tr style="text-align: center;">
      <th>2</th>
      <td>a</td>
      <td>1</td>
      <td>c</td>
      <td>1</td>
      <td>d</td>
      <td>1</td>
      <td>draw</td>
    </tr>
    <tr style="text-align: center;">
      <th>3</th>
      <td>a</td>
      <td>1</td>
      <td>c</td>
      <td>1</td>
      <td>d</td>
      <td>2</td>
      <td>draw</td>
    </tr>
    <tr style="text-align: center;">
      <th>4</th>
      <td>a</td>
      <td>1</td>
      <td>c</td>
      <td>2</td>
      <td>c</td>
      <td>1</td>
      <td>draw</td>
    </tr>
  </tbody>
</table>



```python
data.info() # 查看数据集的基本概况
```

**Out [2]:** 

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 28056 entries, 0 to 28055
Data columns (total 7 columns):
 #   Column  Non-Null Count  Dtype 
---  ------  --------------  ----- 
 0   x1      28056 non-null  object
 1   y1      28056 non-null  int64 
 2   x2      28056 non-null  object
 3   y2      28056 non-null  int64 
 4   x3      28056 non-null  object
 5   y3      28056 non-null  int64 
 6   result  28056 non-null  object
dtypes: int64(3), object(4)
memory usage: 1.5+ MB
```

这里我们可以看到这个数据集中共有28056个实例，其中'x1'，'x2'，'x3'，'result'属性是文本格式，其他属性都是数值类型。后面肯定要对这部分文本类型的属性进行处理，下面我们再来看看这些属性中的具体内容


```python
col_text_index = ['x1','x2','x3','result'] # 将文本类型的属性提取出来
for col_ in col_text_index:
    print(data[col_].value_counts(),'\n---------------')
```

**Out [3]:** 

    d    12136
    c     8726
    b     5316
    a     1878
    Name: x1, dtype: int64 
    ---------------
    h    3616
    g    3599
    f    3582
    e    3576
    a    3468
    b    3438
    c    3409
    d    3368
    Name: x2, dtype: int64 
    ---------------
    h    4848
    g    4600
    f    4352
    e    3450
    a    2920
    d    2796
    b    2700
    c    2390
    Name: x3, dtype: int64 
    ---------------
    fourteen    4553
    thirteen    4194
    twelve      3597
    eleven      2854
    draw        2796
    fifteen     2166
    ten         1985
    nine        1712
    eight       1433
    seven        683
    six          592
    five         471
    sixteen      390
    two          246
    four         198
    three         81
    one           78
    zero          27
    Name: result, dtype: int64 
    ---------------

实际上该数据集各个属性的值都是离散的，每个值都代表该属性中的一种情况，数值的大小在这里其实是没有太大意义的

# 二、数据清洗

依旧按照惯例，对数据集中的文本属性进行数据清洗处理，这里使用 $sklearn.preprocessing.OrdinalEncoder $ 将分类属性中的文本处理为整数。


```python
from sklearn.preprocessing import OrdinalEncoder
enc = OrdinalEncoder()
X = data.drop('result', axis=1)
X_tr = enc.fit_transform(X)
data_tr = pd.DataFrame(X_tr, columns=X.columns, index=X.index) + 1
for attr in ['x1','x2','x3']:
    print(data_tr[attr].value_counts(),'\n---------------')
```

**Out [4]:** 

    4.0    12136
    3.0     8726
    2.0     5316
    1.0     1878
    Name: x1, dtype: int64 
    ---------------
    8.0    3616
    7.0    3599
    6.0    3582
    5.0    3576
    1.0    3468
    2.0    3438
    3.0    3409
    4.0    3368
    Name: x2, dtype: int64 
    ---------------
    8.0    4848
    7.0    4600
    6.0    4352
    5.0    3450
    1.0    2920
    4.0    2796
    2.0    2700
    3.0    2390
    Name: x3, dtype: int64 
    ---------------

这里，顺便将标签也处理一下


```python
label = data['result'].copy()
label[label!='draw'] = -1
label.replace('draw', 1, inplace=True)
label.columns = 'result'
```

这样数据清洗就完毕了，来看看数据的最终效果


```python
data_tr.head()
```

**Out [5]:** 

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: center;">
      <th></th>
      <th>x1</th>
      <th>y1</th>
      <th>x2</th>
      <th>y2</th>
      <th>x3</th>
      <th>y3</th>
    </tr>
  </thead>
  <tbody>
    <tr style="text-align: center;">
      <th>0</th>
      <td>1.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>3.0</td>
      <td>3.0</td>
      <td>2.0</td>
    </tr>
    <tr style="text-align: center;">
      <th>1</th>
      <td>1.0</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>2.0</td>
    </tr>
    <tr style="text-align: center;">
      <th>2</th>
      <td>1.0</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>4.0</td>
      <td>1.0</td>
    </tr>
    <tr style="text-align: center;">
      <th>3</th>
      <td>1.0</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>4.0</td>
      <td>2.0</td>
    </tr>
    <tr style="text-align: center;">
      <th>4</th>
      <td>1.0</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>2.0</td>
      <td>3.0</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>


```python
label.head()
```

**Out [6]:** 


```
0    1
1    1
2    1
3    1
4    1
Name: result, dtype: int64
```

# 三、创建测试集

在进一步处理数据之前，我们需要先生成测试集。接下来，我们将只会用到训练集，测试集只有在最后的模型测试时才会用到。通常来说，测试集取全部数据集的20%即可。这里调用 $sklearn.model\_selection.StratifiedShuffleSplit$ 以分层抽样的形式进行数据集划分


```python
from sklearn.model_selection import StratifiedShuffleSplit
y_tr = label.to_numpy()
ss = StratifiedShuffleSplit(n_splits=1, test_size=0.2, random_state=42)
for train_index, test_index in ss.split(X_tr, y_tr):
    X_trainpre, X_testpre = X_tr[train_index], X_tr[test_index]
    y_train, y_test = y_tr[train_index], y_tr[test_index]
pd.DataFrame(y_train).value_counts()
```

**Out [7]:** 


```
-1    20207
 1     2237
dtype: int64
```

# 四、特征缩放

   特征缩放是数据预处理中重要的环节之一，如果输入的数值属性有非常大的比例差异，往往会导致机器学习算法的性能表现不佳。同比例缩放所有属性的两种常用方法是最小-最大缩放和标准化。其中，标准化方法受到异常值的影响更小，这里尝试使用标准化方法。scikit-learn库中对应的类是 $sklearn.preprocessing.StandardScaler$


```python
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
X_train = scaler.fit_transform(X_trainpre)
X_test = scaler.transform(X_testpre)
pd.DataFrame(X_test, columns=data_tr.columns).head()
```

**Out [8]:** 

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: center;">
      <th></th>
      <th>x1</th>
      <th>y1</th>
      <th>x2</th>
      <th>y2</th>
      <th>x3</th>
      <th>y3</th>
    </tr>
  </thead>
  <tbody>
    <tr style="text-align: center;">
      <th>0</th>
      <td>0.947896</td>
      <td>1.235042</td>
      <td>-0.239276</td>
      <td>-0.226981</td>
      <td>0.415659</td>
      <td>-1.532498</td>
    </tr>
    <tr style="text-align: center;">
      <th>1</th>
      <td>-0.119730</td>
      <td>-0.923108</td>
      <td>0.195286</td>
      <td>1.088712</td>
      <td>-1.748052</td>
      <td>-0.202882</td>
    </tr>
    <tr style="text-align: center;">
      <th>2</th>
      <td>-0.119730</td>
      <td>0.155967</td>
      <td>-0.239276</td>
      <td>1.088712</td>
      <td>-1.748052</td>
      <td>-0.646088</td>
    </tr>
    <tr style="text-align: center;">
      <th>3</th>
      <td>-2.254982</td>
      <td>-0.923108</td>
      <td>-1.108400</td>
      <td>1.088712</td>
      <td>-0.017083</td>
      <td>-1.089293</td>
    </tr>
    <tr style="text-align: center;">
      <th>4</th>
      <td>-1.187356</td>
      <td>-0.923108</td>
      <td>0.195286</td>
      <td>0.211583</td>
      <td>-1.315310</td>
      <td>1.126734</td>
    </tr>
  </tbody>
</table>


# 五、模型选择

   我们选择 RBF 内核的支持向量机，有两个超参数 $C$，$\gamma$ 进行确定。LibSVM 的帮助文档中建议 $C\in [2^{-5}, 2^{15}]$，$\gamma\in[2^{-15},2^{3}]$。这里我们遵循文档规定，在此范围内进行粗略搜索；同时这里采用五折交叉验证方法对超参数的优劣进行判断


```python
from sklearnex import patch_sklearn
patch_sklearn()

from sklearn.model_selection import GridSearchCV
from sklearn.svm import SVC
C_range = np.logspace(-5, 15, 11, base=2)
gamma_range = np.logspace(-15, 3, 10, base=2)
param_grid = dict(gamma=gamma_range, C=C_range)
cv = StratifiedShuffleSplit(n_splits=5, test_size=0.2, random_state=42)
grid = GridSearchCV(SVC(kernel='rbf'), param_grid=param_grid, cv=cv, verbose=3, scoring='accuracy')
grid.fit(X_train, y_train)
print(
    "The best parameters are %s with a score of %0.4f"
    % (grid.best_params_, grid.best_score_)
)
```

**Out [9]:** 


```
Fitting 5 folds for each of 110 candidates, totalling 550 fits
[CV 1/5] END .C=0.03125, gamma=3.0517578125e-05;, score=0.900 total time=   0.4s
[CV 2/5] END .C=0.03125, gamma=3.0517578125e-05;, score=0.900 total time=   0.2s
[CV 3/5] END .C=0.03125, gamma=3.0517578125e-05;, score=0.900 total time=   0.2s
[CV 4/5] END .C=0.03125, gamma=3.0517578125e-05;, score=0.900 total time=   0.2s
[CV 5/5] END .C=0.03125, gamma=3.0517578125e-05;, score=0.900 total time=   0.2s
[CV 1/5] END ..C=0.03125, gamma=0.0001220703125;, score=0.900 total time=   0.2s
[CV 2/5] END ..C=0.03125, gamma=0.0001220703125;, score=0.900 total time=   0.2s
[CV 3/5] END ..C=0.03125, gamma=0.0001220703125;, score=0.900 total time=   0.2s
[CV 4/5] END ..C=0.03125, gamma=0.0001220703125;, score=0.900 total time=   0.2s
.......
[CV 4/5] END ..........C=32768.0, gamma=0.03125;, score=0.996 total time=   6.9s
[CV 5/5] END ..........C=32768.0, gamma=0.03125;, score=0.997 total time=   7.6s
[CV 1/5] END ............C=32768.0, gamma=0.125;, score=0.994 total time=   1.4s
[CV 2/5] END ............C=32768.0, gamma=0.125;, score=0.996 total time=   1.2s
[CV 3/5] END ............C=32768.0, gamma=0.125;, score=0.995 total time=   1.2s
[CV 4/5] END ............C=32768.0, gamma=0.125;, score=0.996 total time=   1.3s
[CV 5/5] END ............C=32768.0, gamma=0.125;, score=0.997 total time=   1.3s
[CV 1/5] END ..............C=32768.0, gamma=0.5;, score=0.999 total time=   0.3s
[CV 2/5] END ..............C=32768.0, gamma=0.5;, score=0.997 total time=   0.3s
The best parameters are {'C': 512.0, 'gamma': 0.5} with a score of 0.9976
```

参数搜索完毕，最好的模型精度可以达到 **99.76%** ，似乎还不错！这就完了？当然没有，也许会存在更优的解，也不一定。花点时间做更进一步搜索也值得，我们可以在前面粗略搜索的最优参数附近作进一步的精细搜索。


```python
# 获取粗略搜索得到的最优参数
C_current_best = grid.best_params_['C']
gamma_current_best = grid.best_params_['gamma']
index_C = np.where(C_range == C_current_best)[0][0]
index_Gamma = np.where(gamma_range == gamma_current_best)[0][0]

# 计算精细搜索的参数范围
n = 10
CScale = np.linspace(-5, 15, 11)
minCScale = (CScale[index_C] + CScale[max(0, index_C-1)]) * 0.5
maxCScale = (CScale[index_C] + CScale[min(len(CScale), index_C+1)]) * 0.5
CRange_finer = np.logspace(minCScale, maxCScale, n+1, base=2)

GammaScale = np.linspace(-15, 3, 10)
minGammaScale = (GammaScale[index_Gamma] + GammaScale[max(0, index_Gamma-1)]) * 0.5
maxGammaScale = (GammaScale[index_Gamma] + GammaScale[min(len(GammaScale), index_Gamma+1)]) * 0.5
GammaRange_finer = np.logspace(minGammaScale, maxGammaScale, n+1, base=2)

param_grid = dict(gamma=GammaRange_finer, C=CRange_finer)
cv = StratifiedShuffleSplit(n_splits=5, test_size=0.2, random_state=42)
grid = GridSearchCV(SVC(kernel='rbf'), param_grid=param_grid, cv=cv, verbose=3, scoring='accuracy')
grid.fit(X_train, y_train)
print(
    "The best parameters are %s with a score of %0.4f"
    % (grid.best_params_, grid.best_score_)
)
```

**Out [10]:** 

```
Fitting 5 folds for each of 121 candidates, totalling 605 fits
[CV 1/5] END ...............C=256.0, gamma=0.25;, score=0.998 total time=   0.4s
[CV 2/5] END ...............C=256.0, gamma=0.25;, score=0.997 total time=   0.4s
[CV 3/5] END ...............C=256.0, gamma=0.25;, score=0.997 total time=   0.4s
[CV 4/5] END ...............C=256.0, gamma=0.25;, score=0.996 total time=   0.3s
[CV 5/5] END ...............C=256.0, gamma=0.25;, score=0.996 total time=   0.4s
...
[CV 1/5] END C=1024.0, gamma=0.8705505632961241;, score=0.998 total time=   0.3s
[CV 2/5] END C=1024.0, gamma=0.8705505632961241;, score=0.998 total time=   0.4s
[CV 3/5] END C=1024.0, gamma=0.8705505632961241;, score=0.997 total time=   0.3s
[CV 4/5] END C=1024.0, gamma=0.8705505632961241;, score=0.997 total time=   0.4s
[CV 5/5] END C=1024.0, gamma=0.8705505632961241;, score=0.999 total time=   0.4s
[CV 1/5] END ...............C=1024.0, gamma=1.0;, score=0.998 total time=   0.3s
[CV 2/5] END ...............C=1024.0, gamma=1.0;, score=0.997 total time=   0.3s
[CV 3/5] END ...............C=1024.0, gamma=1.0;, score=0.997 total time=   0.3s
[CV 4/5] END ...............C=1024.0, gamma=1.0;, score=0.997 total time=   0.4s
[CV 5/5] END ...............C=1024.0, gamma=1.0;, score=0.999 total time=   0.3s
The best parameters are {'C': 256.0, 'gamma': 0.7578582832551991} with a score of 0.9977
```


到这里，我们的参数搜索已经完成。剩下的就是创建 SVM 模型了，这步就很简单了


```python
from sklearn.metrics import accuracy_score

model =SVC(kernel='rbf', gamma=grid.best_params_['gamma'], C=grid.best_params_['C'])
model.fit(X_train, y_train)
pre = model.predict(X_test)
accuracy_score(y_test, pre)
```

**Out [11]:** 

```
测试集准确率：0.998039914468995
```

# 六、代码展示

```python
'''
project: 用 scikit-learn 解决兵王问题
author：Northfourta
date：2022/04/06
dependent libraries：
- numpy
- pandas
- scikit-
'''

import pandas as pd
import numpy as np
from sklearn.preprocessing import OrdinalEncoder
from sklearn.model_selection import StratifiedShuffleSplit
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import GridSearchCV
from sklearn.svm import SVC
from sklearn.metrics import accuracy_score
from sklearnex import patch_sklearn
patch_sklearn() # scikit-learn 加速包

# 加载数据
data = pd.read_csv('krkopt.data', names=['x1','y1','x2','y2','x3','y3','result'])
# 数据清洗
enc = OrdinalEncoder()
X = data.drop('result', axis=1)
X_tr = enc.fit_transform(X)

label = data['result'].copy()
label[label!='draw'] = -1
label.replace('draw', 1, inplace=True)
# 创建测试集
y_tr = label.to_numpy()
ss = StratifiedShuffleSplit(n_splits=1, test_size=0.2, random_state=42)
for train_index, test_index in ss.split(X_tr, y_tr):
    X_trainpre, X_testpre = X_tr[train_index], X_tr[test_index]
    y_train, y_test = y_tr[train_index], y_tr[test_index]
    
# 特征缩放
scaler = StandardScaler()
X_train = scaler.fit_transform(X_trainpre)
X_test = scaler.transform(X_testpre)

# 模型选择
C_range = np.logspace(-5, 15, 11, base=2)
gamma_range = np.logspace(-15, 3, 10, base=2)
param_grid = dict(gamma=gamma_range, C=C_range)
cv = StratifiedShuffleSplit(n_splits=5, test_size=0.2, random_state=42)
grid = GridSearchCV(SVC(kernel='rbf'), param_grid=param_grid, cv=cv, verbose=3, scoring='accuracy')
grid.fit(X_train, y_train)
print(
    "The best parameters are %s with a score of %0.4f"
    % (grid.best_params_, grid.best_score_)
)

# 获取粗略搜索得到的最优参数
C_current_best = grid.best_params_['C']
gamma_current_best = grid.best_params_['gamma']
index_C = np.where(C_range == C_current_best)[0][0]
index_Gamma = np.where(gamma_range == gamma_current_best)[0][0]

# 计算精细搜索的参数范围
n = 10
CScale = np.linspace(-5, 15, 11)
minCScale = (CScale[index_C] + CScale[max(0, index_C-1)]) * 0.5
maxCScale = (CScale[index_C] + CScale[min(len(CScale), index_C+1)]) * 0.5
CRange_finer = np.logspace(minCScale, maxCScale, n+1, base=2)

GammaScale = np.linspace(-15, 3, 10)
minGammaScale = (GammaScale[index_Gamma] + GammaScale[max(0, index_Gamma-1)]) * 0.5
maxGammaScale = (GammaScale[index_Gamma] + GammaScale[min(len(GammaScale), index_Gamma+1)]) * 0.5
GammaRange_finer = np.logspace(minGammaScale, maxGammaScale, n+1, base=2)

param_grid = dict(gamma=GammaRange_finer, C=CRange_finer)
cv = StratifiedShuffleSplit(n_splits=5, test_size=0.2, random_state=42)
grid = GridSearchCV(SVC(kernel='rbf'), param_grid=param_grid, cv=cv, verbose=3, scoring='accuracy')
grid.fit(X_train, y_train)
print(
    "The best parameters are %s with a score of %0.4f"
    % (grid.best_params_, grid.best_score_)
)

# 模型创建
model =SVC(kernel='rbf', gamma=grid.best_params_['gamma'], C=grid.best_params_['C'])
model.fit(X_train, y_train)
pre = model.predict(X_test)
accuracy_score(y_test, pre)
```

