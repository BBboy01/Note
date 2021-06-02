# 数据预处理

## 数据无量纲化

***归一化对异常值十分敏感，因此在做无量纲处理的时候大多选择标准化***

```python
sklearn.preprocessing.MinMaxScaler    # 归一化
sklearn.preprocessing.StandardScaler  # 标准化
```

### 归一化

```python
import pandas as pd
from sklearn.preprocessing import MinMaxScaler

data = [[-1, 2], [-0.5, 6], [0, 10], [1, 18]]

# 实现归一化
scaler = MinMaxScaler()                             # 实例化

result_ = scaler.fit_transform(data)                # 训练和导出结果一步达成
 
original_data = scaler.inverse_transform(result)    # 将归一化后的结果逆转
 
# 使用MinMaxScaler的参数feature_range实现将数据归一化到[0,1]以外的范围中

scaler = MinMaxScaler(feature_range=[5,10])         # 依然实例化
result = scaler.fit_transform(data)                 # fit_transform一步导出结果
 
# 当X中的特征数量非常多的时候，fit会报错并表示，数据量太大了我计算不了
# 此时使用partial_fit作为训练接口
# scaler = scaler.partial_fit(data)
```

### 标准化

```python
from sklearn.preprocessing import StandardScaler
data = [[-1, 2], [-0.5, 6], [0, 10], [1, 18]]
 
scaler = StandardScaler()                       # 实例化
scaler.fit(data)                                # fit，本质是生成均值和方差
 
scaler.mean_                                    # 查看均值的属性mean_
scaler.var_                                     # 查看方差的属性var_
 
x_std = scaler.transform(data)                  # 通过接口导出结果
 
x_std.mean()                                    # 导出的结果是一个数组，用mean()查看均值
x_std.std()                                     # 用std()查看方差
 
scaler.fit_transform(data)                      # 使用fit_transform(data)一步达成结果
 
scaler.inverse_transform(x_std)                 # 使用inverse_transform逆转标准化
```

## 填补缺失值

```python
sklearn.impute.SimpleImputer(missing_values=nan,  # 数据中的缺失值长什么样，默认空值np.nan
                             strategy=’mean’,     # 填补缺失值的策略，默认均值
                             fill_value=None,#startegy为”constant"时可用 可输入字符串或数字表示要填充的值
                            )
```

```python
import pandas as pd
from sklearn.impute import SimpleImputer

data = pd.read_csv(r".\Narrativedata.csv", index_col=0)#index_col=0将第0列作为索引，不写则认为第0列为特征
Age = data.loc[:,"Age"].values.reshape(-1,1)            # sklearn当中特征矩阵必须是二维
imp_mean = SimpleImputer()                              # 实例化，默认均值填补
imp_median = SimpleImputer(strategy="median")           # 用中位数填补
imp_0 = SimpleImputer(strategy="constant",fill_value=0) # 用0填补
 
imp_mean = imp_mean.fit_transform(Age)                  # fit_transform一步完成调取结果
imp_median = imp_median.fit_transform(Age)
imp_0 = imp_0.fit_transform(Age)

# 在这里我们使用中位数填补Age
data.loc[:,"Age"] = imp_median

# 使用众数填补Embarked
Embarked = data.loc[:,"Embarked"].values.reshape(-1,1)
imp_mode = SimpleImputer(strategy = "most_frequent")
data.loc[:,"Embarked"] = imp_mode.fit_transform(Embarked)


# pandas 实现
data_.loc[:,"Age"] = data_.loc[:,"Age"].fillna(data_.loc[:,"Age"].median())
# .fillna 在DataFrame里面直接进行填补
 
data_.dropna(axis=0,inplace=True)
# .dropna(axis=0)删除所有有缺失值的行，.dropna(axis=1)删除所有有缺失值的列
```

## 处理分类型特征：编码与哑变量（将文字性变量转换为数值）

```python
sklearn.preprocessing.LabelEncoder    # 能够将分类转换为分类数值
sklearn.preprocessing.OrdinalEncoder  # 特征专用，能够将分类特征转换为分类数值
sklearn.preprocessing.OneHotEncoder   # 名义变量专用，创建哑变量
sklearn.preprocessing.Binarizer       # 根据阈值将数据二值化
```

### 分类数值化

```python
from sklearn.preprocessing import LabelEncoder
 
y = data_.iloc[:,-1]                        # 要输入的是标签，不是特征矩阵，所以允许一维
 
le = LabelEncoder()                         # 实例化
le = le.fit(y)                              # 导入数据
label = le.transform(y)                     # transform接口调取结果
 
le.classes_                                 # 属性.classes_查看标签中究竟有多少类别

le.fit_transform(y)                         # 也可以直接fit_transform一步到位
 
le.inverse_transform(label)                 # 使用inverse_transform可以逆转
```

### 标签数值化

```python
from sklearn.preprocessing import OrdinalEncoder
 
# 接口categories_对应LabelEncoder的接口classes_，一模一样的功能 
OrdinalEncoder().fit(data_.iloc[:,1:-1]).categories_
 
data_.iloc[:,1:-1] = OrdinalEncoder().fit_transform(data_.iloc[:,1:-1])
```

### 独热编码

|  A   |  B   |  C   |
| ---- | ---- | ---- |
|  0   |  0   |  1   |
|  0   |  1   |  0   |
|  1   |  0   |  0   |

```python
from sklearn.preprocessing import OneHotEncoder
X = data.iloc[:,1:-1]
 
enc = OneHotEncoder(categories='auto').fit(X)
result = enc.transform(X).toarray()

# 直接一步到位
OneHotEncoder(categories='auto').fit_transform(X).toarray()
 
# 依然可以还原
pd.DataFrame(enc.inverse_transform(result))
 
# 返回每一个经过哑变量后生成稀疏矩阵列的名字
enc.get_feature_names()  
 
# axis=1,表示跨行进行合并，也就是将两表左右相连，如果是axis=0，就是将量表上下相连
newdata = pd.concat([data,pd.DataFrame(result)],axis=1)
 
newdata.drop(["Sex","Embarked"],axis=1,inplace=True)
 
newdata.columns = ["Age","Survived","Female","Male","Embarked_C","Embarked_Q","Embarked_S"]
```

### 二值化

```python
from sklearn.preprocessing import Binarizer
X = data_2.iloc[:,0].values.reshape(-1,1)               # 类为特征专用，所以不能使用一维数组
transformer = Binarizer(threshold=30).fit_transform(X)
 
data_2.iloc[:,0] = transformer
```

# 特征选择

## Filter过滤法

```python
sklearn.feature_selection.VarianceThreshold  # 根据方差消除特征
```

### 方差过滤

```python
import pandas as pd
from sklearn.feature_selection import VarianceThreshold

data = pd.read_csv(r".\digit recognizor.csv")
X = data.iloc[:,1:]
data.shape  # (42000, 784)
selector = VarianceThreshold()                      # 实例化，不填参数默认方差为0
X_var0 = selector.fit_transform(X)                  # 获取删除不合格特征之后的新特征矩阵
 
# 也可以直接写成 X = VairanceThreshold().fit_transform(X)
 
X_var0.shape  # (42000, 708)
```

## 相关性过滤

```python
sklearn.feature_selection.SelectKBest  # 选出前K个分数最高的特征的类
sklearn.feature_selection.chi2         # 计算每个非负特征和标签之间的卡方统计量
# 其中F检验分类用于标签是离散型变量的数据，而F检验回归用于标签是连续型变量的数据
sklearn.feature_selection.f_classif    # 捕捉每个特征与标签之间的线性关系
```

### 卡方过滤

卡方检验的原假设是***两组数据是相互独立的***，所以目标是选择***P<0.05***的特征

```python
from sklearn.feature_selection import SelectKBest
from sklearn.feature_selection import chi2
 
# 假设在这里我一直我需要300个特征
X_fschi = SelectKBest(chi2, k=300).fit_transform(X_fsvar, y)
X_fschi.shape  # (42000, 300)
```

```python
chivalue, pvalues_chi = chi2(X_fsvar,y)

# k取多少？我们想要消除所有p值大于设定值，比如0.05或0.01的特征：
k = chivalue.shape[0] - (pvalues_chi > 0.05).sum()
 
# X_fschi = SelectKBest(chi2, k=填写具体的k).fit_transform(X_fsvar, y)
# cross_val_score(RFC(n_estimators=10,random_state=0),X_fschi,y,cv=5).mean()
```

### F检验

```python
from sklearn.feature_selection import f_classif
 
F, pvalues_f = f_classif(X_fsvar,y)
 
k = F.shape[0] - (pvalues_f > 0.05).sum()
 
#X_fsF = SelectKBest(f_classif, k=填写具体的k).fit_transform(X_fsvar, y)
#cross_val_score(RFC(n_estimators=10,random_state=0),X_fsF,y,cv=5).mean()
```

### 互信息法

互信息法是用来捕捉每个特征与标签之间的任意关系（包括线性和非线性关系）的过滤方法。和F检验相似，它既
可以做回归也可以做分类，并且包含两个类feature_selection.mutual_info_classif（互信息分类）和
feature_selection.mutual_info_regression（互信息回归）。这两个类的用法和参数都和F检验一模一样，不过
互信息法比F检验更加强大，F检验只能够找出线性关系，而互信息法可以找出任意关系。

```python
from sklearn.feature_selection import mutual_info_classif as MIC
 
result = MIC(X_fsvar,y)
 
k = result.shape[0] - sum(result <= 0)
 
#X_fsmic = SelectKBest(MIC, k=填写具体的k).fit_transform(X_fsvar, y)
#cross_val_score(RFC(n_estimators=10,random_state=0),X_fsmic,y,cv=5).mean()
```

| 类                     | 说明                                                         | 超参数的选择                                                 |
| :--------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| VarianceThreshold      | 方差过滤，可输入方差阈值，返回方差大于阈值的新特征矩阵       | 看具体数据究竟是含有更多噪声还是更多有效特征 一般就使用0或1来筛选 |
| SelectKBest            | 用来选取K个统计量结果最佳的特征，生成符合统计量要求的新特征矩阵 | 看配合使用的统计量                                           |
| chi2                   | 卡方检验，专用于分类算法，捕捉相关性                         | 追求p小于显著性水平的特征                                    |
| f_classif              | F检验分类，只能捕捉线性相关性要求数据服从正态分布            | 追求p小于显著性水平的特征                                    |
| f_regression           | F检验回归，只能捕捉线性相关性要求数据服从正态分布            | 追求p小于显著性水平的特征                                    |
| mutual_info_classif    | 互信息分类，可以捕捉任何相关性,不能用于稀疏矩阵              | 追求互信息估计大于0的特征                                    |
| mutual_info_regression | 互信息回归，可以捕捉任何相关性                               | 追求互信息估计大于0的特征                                    |

### Embedded嵌入法（看课件）

# PCA特征选择

```python
sklearn.decomposition.PCA
```

特征工程中有三种方式：特征提取，特征创造和特征选择。仔细观察上面的降维例子和上周我们讲解过的特征
选择，你发现有什么不同了吗?
特征选择是从已存在的特征中选取携带信息最多的，选完之后的特征依然具有可解释性，我们依然知道这个特
征在原数据的哪个位置，代表着原数据上的什么含义。
**而PCA，是将已存在的特征进行压缩，降维完毕后的特征不是原本的特征矩阵中的任何一个特征，而是通过某**
**些方式组合起来的新特征。通常来说，在新的特征矩阵生成之前，我们无法知晓PCA都建立了怎样的新特征向**
**量，新特征矩阵生成之后也不具有可读性，我们无法判断新特征矩阵的特征是从原数据中的什么特征组合而**
**来，新特征虽然带有原始数据的信息，却已经不是原数据上代表着的含义了。以PCA为代表的降维算法因此是**
**特征创造（feature creation，或feature construction）的一种。**
可以想见，PCA一般不适用于探索特征和标签之间的关系的模型（如线性回归），因为***无法解释的新特征和标
签之间的关系不具有意义***。在线性回归模型中，我们使用特征选择。

## 获取指定数量的特征

```python
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.datasets import load_iris
from sklearn.decomposition import PCA

iris = load_iris()
y = iris.target
X = iris.data
X.shape                             # (150, 4)

pca = PCA(n_components=2)           # 实例化
pca = pca.fit(X)                    # 拟合模型
X_dr = pca.transform(X)             # 获取新矩阵 也可以fit_transform一步到位
```

```python
#属性explained_variance_，查看降维后每个新特征向量上所带的信息量大小（可解释性方差的大小）
pca.explained_variance_#查看方差是否从大到小排列，第一个最大，依次减小   array([4.22824171, 0.24267075])
 
#属性explained_variance_ratio，查看降维后每个新特征向量所占的信息量占原始数据总信息量的百分比
#又叫做可解释方差贡献率
pca.explained_variance_ratio_#array([0.92461872, 0.05306648])
#大部分信息都被有效地集中在了第一个特征上
 
pca.explained_variance_ratio_.sum()#0.977685206318795
```

## 画出累计方差贡献率

```python

## 按信息量占比选超参数

输入[0,1]之间的浮点数，并且让参数**svd_solver =='full'**，表示希望降维后的总解释性方差占比大于n_components
指定的百分比，即是说，希望保留百分之多少的信息量。比如说，如果我们希望保留97%的信息量，就可以输入
n_components = 0.97，PCA会自动选出能够让保留的信息量超过97%的特征数量

​```python
pca_f = PCA(n_components=0.97,svd_solver="full")#svd_solver="full"不能省略
pca_f = pca_f.fit(X)
X_f = pca_f.transform(X)
X_f 
pca_f.explained_variance_ratio_#array([0.92461872, 0.05306648])
```

## 获取SVD种的数据

当V(k,n)是数字
时，我们无法判断V(k,n)和原有的特征究竟有着怎样千丝万缕的数学联系。但是，如果原特征矩阵是图像，V(k,n)这
个空间矩阵也可以被可视化

```python
pca = PCA(150).fit(X)#这里X = faces.data，不是faces.images.shape ,因为sklearn只接受2维数组降，不接受高维数组降
# x_dr = pca.transform(X)
# x_dr.shape#(1277,150)

V = pca.components_  # 新特征空间
```

## 转换结果

inverse_transform并没有实现数据的完全逆转。
这是因为，在降维的时候，部分信息已经被舍弃了，X_dr 中往往不会包含原数据100%的信息，所以在逆转的时
候，即便维度升高，原数据中已经被舍弃的信息也不可能再回来了。所以，**降维不是完全可逆的**

```python
pca = PCA(150)               # 实例化
X_dr = pca.fit_transform(X)  # 拟合+提取结果
X_inverse = pca.inverse_transform(X_dr)
```

# K-Means聚类

默认使用K-Means++

```python
sklearn.cluster.KMeans
```

```python
from sklearn.cluster import KMeans
from sklearn.datasets import make_blobs

X, y = make_blobs(n_samples=500,n_features=2,centers=4,random_state=1)
X.shape  #（500， 2）

n_clusters = 3
cluster = KMeans(n_clusters=n_clusters,random_state=0).fit(X)  # 只接受二维数据

# 重要属性Labels_，查看聚好的类别，每个样本所对应的类
y_pred = cluster.labels_

# 重要属性cLuster_centers_，查看质心
centroid = cluster.cluster_centers_

#重要属性inertia_，查看总距离平方和（应该不咋地用）
inertia = cluster.inertia_
inertia
```

当数据量太大时 可以使用一部分数据进行训练 另一部分用来预测

```python
cluster_smallsub = KMeans(n_clusters=n_clusters,random_state=0).fit(X[:200])
y_pred_ = cluster_smallsub.predict(X)  # 返回预测样本对应的类
```

## 聚类算法的模型评估指标

```python
sklearn.metrics.silhouette_score        # 一个数据集中，所有样本的轮廓系数的均值
sklearn.metrics.silhouette_samples      # 数据集中每个样本自己的轮廓系数
sklearn.metrics.calinski_harabaz_score  # Calinski-harabaz指数越高越好
```

很容易理解轮廓系数范围是(-1,1)，其中值越接近1表示样本与自己所在的簇中的样本很相似，并且与其他簇中的样本不相似，当样本点与簇外的样本更相似的时候，轮廓系数就为负。当轮廓系数为0时，则代表两个簇中的样本相似度一致，两个簇本应该是一个簇。**可以总结为轮廓系数越接近于1越好，负数则表示聚类效果非常差。**
如果一个簇中的大多数样本具有比较高的轮廓系数，则簇会有较高的总轮廓系数，则整个数据集的平均轮廓系数越高，则聚类是合适的。如果许多样本点具有低轮廓系数甚至负值，则聚类是不合适的，聚类的超参数K可能设定得太大或者太小。

```python
from sklearn.metrics import silhouette_score
from sklearn.metrics import silhouette_samples
from sklearn.metrics import calinski_harabaz_score

silhouette_score(X, y_pred)
silhouette_samples(X, y_pred)

calinski_harabaz_score(X, y_pred)
```

## 基于轮廓系数选择n

主要看每一类的轮廓系数对均值贡献的多少

```python
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_samples, silhouette_score

import matplotlib.pyplot as plt
import matplotlib.cm as cm
import numpy as np

for n_clusters in [2,3,4,5,6,7]:
    n_clusters = n_clusters
    fig, (ax1, ax2) = plt.subplots(1, 2)
    fig.set_size_inches(18, 7)
    ax1.set_xlim([-0.1, 1])
    ax1.set_ylim([0, X.shape[0] + (n_clusters + 1) * 10])
    clusterer = KMeans(n_clusters=n_clusters, random_state=10).fit(X)
    cluster_labels = clusterer.labels_
    silhouette_avg = silhouette_score(X, cluster_labels)
    print("For n_clusters =", n_clusters,
          "The average silhouette_score is :", silhouette_avg)
    sample_silhouette_values = silhouette_samples(X, cluster_labels)
    y_lower = 10
    for i in range(n_clusters):
        ith_cluster_silhouette_values = sample_silhouette_values[cluster_labels == i]
        ith_cluster_silhouette_values.sort()
        size_cluster_i = ith_cluster_silhouette_values.shape[0]
        y_upper = y_lower + size_cluster_i
        color = cm.nipy_spectral(float(i)/n_clusters)
        ax1.fill_betweenx(np.arange(y_lower, y_upper)
                          ,ith_cluster_silhouette_values
                          ,facecolor=color
                          ,alpha=0.7
                         )
        ax1.text(-0.05
                 , y_lower + 0.5 * size_cluster_i
                 , str(i))
        y_lower = y_upper + 10

    ax1.set_title("The silhouette plot for the various clusters.")
    ax1.set_xlabel("The silhouette coefficient values")
    ax1.set_ylabel("Cluster label")
    ax1.axvline(x=silhouette_avg, color="red", linestyle="--")
    ax1.set_yticks([])
    ax1.set_xticks([-0.1, 0, 0.2, 0.4, 0.6, 0.8, 1])

    colors = cm.nipy_spectral(cluster_labels.astype(float) / n_clusters)
    ax2.scatter(X[:, 0], X[:, 1]
                ,marker='o'
                ,s=8
                ,c=colors
               )
    centers = clusterer.cluster_centers_
    # Draw white circles at cluster centers
    ax2.scatter(centers[:, 0], centers[:, 1], marker='x',
                c="red", alpha=1, s=200)
    
    ax2.set_title("The visualization of the clustered data.")
    ax2.set_xlabel("Feature space for the 1st feature")
    ax2.set_ylabel("Feature space for the 2nd feature")

    plt.suptitle(("Silhouette analysis for KMeans clustering on sample data "
                  "with n_clusters = %d" % n_clusters),
                 fontsize=14, fontweight='bold')
    plt.show()
```

## 图片压缩（看课件）

# 逻辑回归（看课件）

分类算法

```python
sklearn.linear_model.LogisticRegression
```

但是更有效的方法，毫无疑问会是我们的embedded嵌入法。我们已经说明了，由于L1正则化会使得部分特征对应的参数为0，因此L1正则化可以用来做特征选择，结合嵌入法的模块SelectFromModel，我们可以很容易就筛选出让模型十分高效的特征。注意，此时我们的目的是，尽量保留原数据上的信息，让模型在降维后的数据上的拟合效果保持优秀，因此我们不考虑训练集测试集的问题，把所有的数据都放入模型进行降维。

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import LogisticRegression as LR
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score  # 精确性分数
 
data = load_breast_cancer()  # 乳腺癌数据集
X = data.data
y = data.target 
X.data.shape  # (569, 30)

# l1,l2 正则化做惩罚项方式  C 正则化强度
lrl1 = LR(penalty="l1",solver="liblinear",C=0.5,max_iter=1000) 
lrl2 = LR(penalty="l2",solver="liblinear",C=0.5,max_iter=1000) 

# 逻辑回归的重要属性coef_，查看每个特征所对应的参数
lrl1 = lrl1.fit(X,y)
lrl1.coef_

(lrl1.coef_ != 0).sum(axis=1)  # array([10])  30个特征中有10个特征的系数不为0

lrl2 = lrl2.fit(X,y)
lrl2.coef_

l1 = []
l2 = []
l1test = []
l2test = []
 
Xtrain, Xtest, Ytrain, Ytest = train_test_split(X,y,test_size=0.3,random_state=420)
 
for i in np.linspace(0.05,1.5,19):
    lrl1 = LR(penalty="l1",solver="liblinear",C=i,max_iter=1000)
    lrl2 = LR(penalty="l2",solver="liblinear",C=i,max_iter=1000)
    
    lrl1 = lrl1.fit(Xtrain,Ytrain)
    l1.append(accuracy_score(lrl1.predict(Xtrain),Ytrain))
    l1test.append(accuracy_score(lrl1.predict(Xtest),Ytest))
    lrl2 = lrl2.fit(Xtrain,Ytrain)
    l2.append(accuracy_score(lrl2.predict(Xtrain),Ytrain))
    l2test.append(accuracy_score(lrl2.predict(Xtest),Ytest))
 
graph = [l1,l2,l1test,l2test]
color = ["green","black","lightgreen","gray"]
label = ["L1","L2","L1test","L2test"]    
 
plt.figure(figsize=(6,6))
for i in range(len(graph)):
    plt.plot(np.linspace(0.05,1.5,19),graph[i],color[i],label=label[i])
plt.legend(loc=4) #图例的位置在哪里?4表示，右下角
plt.show()
```

# 回归大家族(线性 岭 Lasso 多项式)

```python
sklearn.linear_model.LinearRegression     # 线性回归
sklearn.linear_model.Ridge                # 岭回归
sklearn.linear_model.Lasso                # Lasso回归
sklearn.preprocessing.PolynomialFeatures  # 多项式回归
```

评估指标MSE

均方误差，本质是在RSS的基础上除以了样本总量，得到了每个样本量上的平均误差。有了平均误差，我们就可以将平均误差和我们的标签的取值范围在一起比较，以此获得一个较为可靠的评估依据。在sklearn当中，我们有两种方式调用这个评估指标，一种是使用sklearn专用的模型评估模块metrics里的类mean_squared_error，另一种是调用交叉验证的类cross_val_score并使用里面的scoring参数来设置使用均方误差。

R^2 越接近1越好

```python
from sklearn.metrics import mean_squared_error as MSE
MSE(yhat,Ytest)
cross_val_score(reg,X,y,cv=10,scoring="neg_mean_squared_error").mean()
```

多重共线性是一种统计现象，是指线性模型中的特征（解释变量）之间由于存在精确相关关系或高度相关关系，多重共线性的存在会使模型无法建立，或者估计失真。多重共线性使用指标方差膨胀因子（variance inflation factor，VIF）来进行衡量（from statsmodels.stats.outliers_influence import variance_inflation_factor），通常当我们提到“共线性”，都特指多重共线性。相关性是衡量两个或多个变量一起波动的程度的指标，它可以是正的，负的或者0。当我们说变量之间具有相关性，通常是指线性相关性，线性相关一般由皮尔逊相关系数进行衡量，非线性相关可以使用斯皮尔曼相关系数或者互信息法进行衡量。

## 岭回归

在统计学中，我们会通过VIF或者各种检验来判断数据是否存在共线性，然而在机器学习中，我们可以使用模型来判断——如果一个数据集在岭回归中使用各种正则化参数取值下模型表现没有明显上升（比如出现持平或者下降），则说明数据没有多重共线性，顶多是特征之间有一些相关性。反之，如果一个数据集在岭回归的各种正则化参数取值下表现出明显的上升趋势，则说明数据存在多重共线性。

```python
reg = LR().fit(Xtrain, Ytrain)
r2 = reg.score(Xtest,Ytest)  # 0.6043668160178817
reg = Ridge(alpha=1).fit(Xtrain,Ytrain)
reg.score(Xtest,Ytest)       # 0.6043610352312276 加利佛尼亚房屋价值数据集中应该不是共线性问题

#交叉验证下，与线性回归相比，岭回归的结果如何变化？
#细化一下学习曲线
alpharange = np.arange(1,201,10)
ridge, lr = [], []
for alpha in alpharange:
    reg = Ridge(alpha=alpha)
    linear = LinearRegression()
    regs = cross_val_score(reg,X,y,cv=5,scoring = "r2").mean()
    linears = cross_val_score(linear,X,y,cv=5,scoring = "r2").mean()
    ridge.append(regs)
    lr.append(linears)
plt.plot(alpharange,ridge,color="red",label="Ridge")
plt.plot(alpharange,lr,color="orange",label="LR")
plt.title("Mean")
plt.legend()
plt.show()

#模型方差如何变化？
alpharange = np.arange(1,1001,100)
ridge, lr = [], []
for alpha in alpharange:
    reg = Ridge(alpha=alpha)
    linear = LinearRegression()
    varR = cross_val_score(reg,X,y,cv=5,scoring="r2").var()
    varLR = cross_val_score(linear,X,y,cv=5,scoring="r2").var()
    ridge.append(varR)
    lr.append(varLR)
plt.plot(alpharange,ridge,color="red",label="Ridge")
plt.plot(alpharange,lr,color="orange",label="LR")
plt.title("Variance")
plt.legend()
plt.show()
'''
可以发现，模型的方差上升快速，不过方差的值本身很小，其变化不超过上升部分的1/3，因此只要噪声的状况维
持恒定，模型的泛化误差可能还是一定程度上降低了的
'''
```

### 带交叉验证的岭回归

使用交叉验证来选择最佳的正则化系数alpha

```python
import numpy as np
import pandas as pd
from sklearn.linear_model import RidgeCV, LinearRegression
from sklearn.model_selection import train_test_split as TTS
from sklearn.datasets import fetch_california_housing as fch
import matplotlib.pyplot as plt

housevalue = fch()

X = pd.DataFrame(housevalue.data)
y = housevalue.target
X.columns = ["住户收入中位数","房屋使用年代中位数","平均房间数目"
            ,"平均卧室数目","街区人口","平均入住率","街区的纬度","街区的经度"]
Ridge_ = RidgeCV(alphas=np.arange(1,1001,100)
                 #,scoring="neg_mean_squared_error"
                 ,store_cv_values=True
                 #,cv=5
                ).fit(X, y)

# 无关交叉验证的岭回归结果
Ridge_.score(X,y)  # 0.6060251767338429
# 调用所有交叉验证的结果
Ridge_.cv_values_
# 进行平均后可以查看每个正则化系数取值下的交叉验证结果
Ridge_.cv_values_.mean(axis=0)
# 查看被选择出来的最佳正则化系数
Ridge_.alpha_      # 101
```

## Lasso回归

Lasso的核心作用：特征选择

Lasso不是从根本上解决多重共线性问题，而是限制多重共线性带来的影响

Lasso所带的L1正则项对于系数的惩罚要重得多，并且它会将系数压缩至0，因此可以被用来做特征选择。也因此，我们往往让Lasso的正则化系数在很小的空间中变动，以此来寻找最佳的正则化系数。

### 带交叉验证的Lasso回归

```python
from sklearn.linear_model import LassoCV

# 自己建立Lasso进行alpha选择的范围
alpharange = np.logspace(-10, -2, 200,base=10)

# 其实是形成10为底的指数函数 10**(-10)到10**(-2)次方

lasso_ = LassoCV(alphas=alpharange  # 自行输入的alpha的取值范围
                ,cv=5               # 交叉验证的折数
                ).fit(Xtrain, Ytrain)

# 查看被选择出来的最佳正则化系数
lasso_.alpha_

# 调用所有交叉验证的结果
lasso_.mse_path_

lasso_.mse_path_.shape              # 返回每个alpha下的五折交叉验证结果

lasso_.mse_path_.mean(axis=1)       # 有注意到在岭回归中我们的轴向是axis=0吗？

# 在岭回归当中，我们是留一验证，因此我们的交叉验证结果返回的是，每一个样本在每个alpha下的交叉验证结果
# 因此我们要求每个alpha下的交叉验证均值，就是axis=0，跨行求均值
# 而在这里，我们返回的是，每一个alpha取值下，每一折交叉验证的结果
# 因此我们要求每个alpha下的交叉验证均值，就是axis=1，跨列求均值

# 最佳正则化系数下获得的模型的系数结果
lasso_.coef_

lasso_.score(Xtest,Ytest)
```

## 多项式回归

对特征进行升维

```python
from sklearn.preprocessing import PolynomialFeatures
import numpy as np

X = np.arange(1,4).reshape(-1,1)

# 二次多项式，参数degree控制多项式的次方
poly = PolynomialFeatures(degree=2)
# 接口transform直接调用
X_ = poly.fit_transform(X)
X_.shape  # (3, 3)

# 三次多项式，不带与截距项相乘的x0
# 对于多项式回归来说，我们已经为线性回归准备好了x0，但是线性回归并不知道
x = PolynomialFeatures(degree=3,include_bias=False).fit_transform(X)
x.shape    # (3, 3)
xxx = PolynomialFeatures(degree=3).fit_transform(X)
xxx.shape  # (3, 4)

PolynomialFeatures(degree=2,interaction_only=True).fit_transform(X)
# 对比之下，当interaction_only为True的时候，只生成交互项

poly = PolynomialFeatures(degree=5).fit(X)
# 重要接口get_feature_names 查看新特征究竟是由原特征矩阵中的什么特征组成的
poly.get_feature_names()
```

```python
from sklearn.datasets import fetch_california_housing as fch
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import PolynomialFeatures
import pandas as pd

housevalue = fch()
X = pd.DataFrame(housevalue.data)
y = housevalue.target
X.columns = ["住户收入中位数","房屋使用年代中位数","平均房间数目"
            ,"平均卧室数目","街区人口","平均入住率","街区的纬度","街区的经度"]

poly = PolynomialFeatures(degree=4).fit(X)
poly.get_feature_names(X.columns)

X_ = poly.transform(X)
# 在这之后，我们依然可以直接建立模型，然后使用线性回归的coef_属性来查看什么特征对标签的影响最大
reg = LinearRegression().fit(X_,y)
coef = reg.coef_
# 放到dataframe中进行排序
coeff = pd.DataFrame([poly.get_feature_names(X.columns),reg.coef_.tolist()]).T
coeff.columns = ["feature","coef"]
coeff.sort_values(by="coef")

#顺便可以查看一下多项式变化之后，模型的拟合效果如何了
poly = PolynomialFeatures(degree=4).fit(X)
X_ = poly.transform(X)
reg_ = LinearRegression().fit(X,y)
reg_.score(X,y)   # 0.6062326851998051
reg.score(X_,y)  # 0.7452006076530354
```



# SVM(二分类)有监督

**1.线性核，尤其是多项式核函数在高次项时计算非常缓慢**

**2.rbf和多项式核函数都不擅长处理量纲不统一的数据集**

**核函数主要选 rbf 和 liner**

**rbf先调gamma 再调C**

**参数C**：浮点数，默认1，必须大于等于0
松弛系数的惩罚项系数。如果C值设定比较大，那SVC可能会选择边际较小的，能够更好地分类所有训练点的决策边界，不过模型的训练时间也会更长。如果C的设定值较小，那SVC会尽量最大化边界，决策功能会更简单，但代价是训练的准确度。换句话说，C在SVM中的影响就像正则化参数对逻辑回归的

**参数class_weight**：可输入字典或者"balanced”，可不填，默认None 对SVC，将类i的参数C设置为class_weight [i] * C。如果没有给出具体的class_weight，则所有类都被假设为占有相同的权重1，模型会根据数据原本的状况去训练。如果希望改善样本不均衡状况，请输入形如{"标签的值1"：权重1，"标签的值2"：权重2}的字典，则参数C将会自动被设为：标签的值1的C：权重1 * C，标签的值2的C：权重2*C
或者，可以使用“balanced”模式，这个模式使用y的值自动调整与输入数据中的类频率成反比的权重为
n_samples/(n_classes * np.bincount(y))

关键概念：硬间隔与软间隔
当两组数据是完全线性可分，我们可以找出一个决策边界使得训练集上的分类误差为0，这两种数据就被称为是存在”硬间隔“的。当两组数据几乎是完全线性可分的，但决策边界在训练集上存在较小的训练误差，这两种数据就被称为是存在”软间隔“。

```python
sklearn.svm.SVC
```

```python
clf.predict(X)
#根据决策边界，对X中的样本进行分类，返回的结构为n_samples
 
clf.score(X,y)
#返回给定测试数据和标签的平均准确度
 
clf.support_vectors_
#返回支持向量坐标
 
clf.n_support_  # array([2, 1])
#返回每个类中支持向量的个数
```

|  输入   |    含义    | 解决问题 | 参数 gamma | 参数 degree | 参数 degree |
| :-----: | :--------: | :------: | :--------: | :---------: | :---------: |
|  liner  |   线性核   |   线性   |     No     |     No      |     No      |
|  poly   |  多项式核  |  偏线性  |    Yes     |     Yes     |     Yes     |
| sigmoid | 双曲正切核 |  非线性  |    Yes     |     No      |     Yes     |
| **rbf** | 离斯径向基 | 偏非线性 |    Yes     |     No      |     No      |

## 线性分类

```python
import matplotlib.pyplot as plt
import numpy as np
from sklearn.datasets import make_blobs
from sklearn.svm import SVC

X,y = make_blobs(n_samples=50, centers=2, random_state=0,cluster_std=0.6)

plt.scatter(X[:,0],X[:,1],c=y,s=50,cmap="rainbow")
ax = plt.gca() #获取当前的子图，如果不存在，则创建新的子图

#建模，通过fit计算出对应的决策边界
clf = SVC(kernel = "linear").fit(X,y)#计算出对应的决策边界

#获取平面上两条坐标轴的最大值和最小值
xlim = ax.get_xlim()
ylim = ax.get_ylim()
 
#在最大值和最小值之间形成30个规律的数据
axisx = np.linspace(xlim[0],xlim[1],30)
axisy = np.linspace(ylim[0],ylim[1],30)
 
axisy,axisx = np.meshgrid(axisy,axisx)
#我们将使用这里形成的二维数组作为我们contour函数中的X和Y
#使用meshgrid函数将两个一维向量转换为特征矩阵
#核心是将两个特征向量广播，以便获取y.shape * x.shape这么多个坐标点的横坐标和纵坐标
 
xy = np.vstack([axisx.ravel(), axisy.ravel()]).T
#其中ravel()是降维函数，vstack能够将多个结构一致的一维数组按行堆叠起来
#xy就是已经形成的网格，它是遍布在整个画布上的密集的点

Z = clf.decision_function(xy).reshape(axisx.shape)
#重要接口decision_function，返回每个输入的样本所对应的到决策边界的距离
#然后再将这个距离转换为axisx的结构，这是由于画图的函数contour要求Z的结构必须与X和Y保持一致

#首先要有散点图
plt.scatter(X[:,0],X[:,1],c=y,s=50,cmap="rainbow")
ax = plt.gca() #获取当前的子图，如果不存在，则创建新的子图
#画决策边界和平行于决策边界的超平面
ax.contour(axisx,axisy,Z
           ,colors="k"
           ,levels=[-1,0,1] #画三条等高线，分别是Z为-1，Z为0和Z为1的三条线
           ,alpha=0.5#透明度
           ,linestyles=["--","-","--"])
 
ax.set_xlim(xlim)#设置x轴取值
ax.set_ylim(ylim)
```

## 非线性分类

```python
from sklearn.svm import SVC
import matplotlib.pyplot as plt
import numpy as np
 
from sklearn.datasets import make_circles
X,y = make_circles(100, factor=0.1, noise=.1)
 
def plot_svc_decision_function(model,ax=None):
    if ax is None:
        ax = plt.gca()
    xlim = ax.get_xlim()
    ylim = ax.get_ylim()
    
    x = np.linspace(xlim[0],xlim[1],30)
    y = np.linspace(ylim[0],ylim[1],30)
    Y,X = np.meshgrid(y,x) 
    xy = np.vstack([X.ravel(), Y.ravel()]).T
    P = model.decision_function(xy).reshape(X.shape)
    
    ax.contour(X, Y, P,colors="k",levels=[-1,0,1],alpha=0.5,linestyles=["--","-","--"])
    ax.set_xlim(xlim)
    ax.set_ylim(ylim)

clf = SVC(kernel = "rbf").fit(X,y)
plt.scatter(X[:,0],X[:,1],c=y,s=50,cmap="rainbow")
plot_svc_decision_function(clf)
```

学习曲线看课件

## 模型评估指标

**准确率**Accuracy就是所有预测正确的所有样本除以总样本，通常来说越接近1越好

**精确度**Precision，又叫查准率，表示所有被我们预测为是少数类的样本中，真正的少数类所占的比例，精确度高对少数类的预测越精确。精确度越低，则代表我们误伤了过多的多数类

**召回率**Recall，又被称为敏感度(sensitivity)，真正率，查全率，表示所有真实为1的样本中，被我们预测正确的样
本所占的比例。在支持向量机中，召回率可以被表示为，决策边界上方的所有红色点占全部样本中的红色点的比
例。召回率越高，代表我们尽量捕捉出了越多的少数类，召回率越低，代表我们没有捕捉出足够的少数类。

**特异度**Specificity表示所有真实为0的样本中，被正确预测为0的样本所占的比例。在支持向量机中，可以形象地
表示为，决策边界下方的点占所有紫色点的比例。

| 类                                     | 含义                  |
| -------------------------------------- | --------------------- |
| sklearn.metrics.confusion_matrix       | 混淆矩阵              |
| sklearn.metrics.accuracy_score         | 准确率accuracy        |
| sklearn.metrics.precision_score        | 精确度precision       |
| sklearn.metrics.recall_score           | 召回率recall          |
| sklearn.metrics.precision_recall_curve | 精确度-召回率平衡曲线 |
| sklearn.metrics.f1_score               | F1 measure            |

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn import svm
from sklearn.datasets import make_blobs
from sklearn.linear_model import LogisticRegression as LogiR
from sklearn.metrics import confusion_matrix as CM, precision_score as P, recall_score as R

class_1_ = 7
class_2_ = 4
centers_ = [[0.0, 0.0], [1,1]]
clusters_std = [0.5, 1]
X_, y_ = make_blobs(n_samples=[class_1_, class_2_], 
                    centers=centers_, 
                    cluster_std=clusters_std,
                    random_state=0, shuffle=False
                   )
clf_lo = LogiR().fit(X_,y_)
prob = clf_lo.predict_proba(X_)
prob = pd.DataFrame(prob)
prob.columns = ["0","1"]
# 手动调节阈值，来改变我们的模型效果
for i in range(prob.shape[0]):
    if prob.loc[i,"1"] > 0.5:
        prob.loc[i,"pred"] = 1
    else:
        prob.loc[i,"pred"] = 0
prob["y_true"] = y_
prob = prob.sort_values(by="1",ascending=False)
# 混淆矩阵
CM(prob.loc[:,"y_true"],prob.loc[:,"pred"],labels=[1,0])  # array([[2, 2], [1, 6]], dtype=int64)
# 召回率
P(prob.loc[:,"y_true"],prob.loc[:,"pred"],labels=[1,0])   # 0.6666666666666666
# 精确度
R(prob.loc[:,"y_true"],prob.loc[:,"pred"],labels=[1,0])   # 0.5
```

**predict_proba** 返回样本属于哪个类的概率

设置为True则会启动，启用之后，SVC的接口predict_proba和predict_log_proba将生效。在二分类情况下，SVC将使用Platt缩放来生成概率，即在decision_function生成的距离上进行Sigmoid压缩，并附加训练数据的交叉验证拟合，来生成类逻辑回归的SVM分数。

有可能出现predict_proba返回的概率小于0.5，但样本依旧被标记为正类的情况出现，毕竟支持向量机本身并不依赖于概率来完成自己的分类

```python
class_1 = 500 #类别1有500个样本
class_2 = 50 #类别2只有50个
centers = [[0.0, 0.0], [2.0, 2.0]] #设定两个类别的中心
clusters_std = [1.5, 0.5] #设定两个类别的方差，通常来说，样本量比较大的类别会更加松散
X, y = make_blobs(n_samples=[class_1, class_2],
                  centers=centers,
                  cluster_std=clusters_std,
                  random_state=0, shuffle=False)

clf_proba = svm.SVC(kernel="linear",C=1.0,probability=True).fit(X,y)
# 生成的各类标签下的概率
clf_proba.predict_proba(X).shape      # (550, 2)
# 返回的值也因此被我们认为是SVM中的置信度
clf_proba.decision_function(X).shape  # (550, )
```

**ROC曲线**，全称The Receiver Operating Characteristic Curve，译为受试者操作特性曲线。这是一条以不同阈值下的假正率FPR为横坐标，不同阈值下的召回率Recall为纵坐标的曲线

ROC是一条以不同阈值下的假正率FPR为横坐标，不同阈值下的召回率Recall为纵坐标的曲线

我们建立ROC曲线的根本目的是找寻Recall和FPR之间的平衡，让我们能够衡量模型在尽量捕捉少数类的时候，误伤多数类的情况会如何变化。横坐标是FPR，代表着模型将多数类判断错误的能力，纵坐标Recall，代表着模型捕捉少数类的能力，所以ROC曲线代表着，随着Recall的不断增加，FPR如何增加

中间的虚线代表着，当recall增加1%，我们的FPR也增加1%，也就是说，我们每捕捉出一个少数类，就会有一个多数类被判错，这种情况下，模型的效果就不好，这种模型捕获少数类的结果，会让许多多数类被误伤，从而增加我们的成本。ROC曲线通常都是凸型的。对于一条凸型ROC曲线来说，曲线越靠近左上角越好，越往下越糟糕，曲线如果在虚线的下方，则证明模型完全无法使用。但是它也有可能是一条凹形的ROC曲线。对于一条凹型ROC曲线来说，应该越靠近右下角越好，凹形曲线代表模型的预测结果与真实情况完全相反，那也不算非常糟糕，只要我们手动将模型的结果逆转，就可以得到一条左上方的弧线了。最糟糕的就是，无论曲线是凹形还是凸型，曲线位于图像中间，和虚线非常靠近，那我们拿它无能为力

AUC时ROC曲线和X轴的面积。**AUC面积越大越好**

### 绘制ROC曲线  计算AUC面积

```python
from sklearn.datasets import make_blobs
from sklearn.svm import SVC
from sklearn.metrics import roc_curve
from sklearn.metrics import roc_auc_score as AUC

class_1 = 500                       # 类别1有500个样本
class_2 = 50                        # 类别2只有50个
centers = [[0.0, 0.0], [2.0, 2.0]]  # 设定两个类别的中心
clusters_std = [1.5, 0.5]           # 设定两个类别的方差，通常来说，样本量比较大的类别会更加松散
X, y = make_blobs(n_samples=[class_1, class_2],
                  centers=centers,
                  cluster_std=clusters_std,
                  random_state=0, shuffle=False)
clf_proba = svm.SVC(kernel="linear",C=1.0,probability=True).fit(X,y)

FPR, recall, thresholds = roc_curve(y,
                                    clf_proba.decision_function(X), 
                                    pos_label=1  # 表示被认为是正类样本的类别 默认为None
                                   )
thresholds  # 此时的threshold就不是一个概率值，而是距离值中的阈值了，所以它可以大于1，也可以为负

area = AUC(y,clf_proba.decision_function(X))     # 计算AUC面积

plt.figure()
plt.plot(FPR, recall, color='red',
         label='ROC curve (area = %0.2f)' % area)
plt.plot([0, 1], [0, 1], color='black',linestyle='--')
plt.xlim([-0.05, 1.05])
plt.ylim([-0.05, 1.05])
plt.xlabel('False Positive Rate')
plt.ylabel('Recall')
plt.title('Receiver operating characteristic example')
plt.legend(loc="lower right")
plt.show()
```

### 通过ROC曲线选最佳阈值

ROC曲线反应的是recall增加的时候FPR如何变化，也就是当模型捕获少数类的能力变强的时候，会误伤多数类的情况是否严重。我们的希望是，模型在捕获少数类的能力变强的时候，尽量不误伤多数类，也就是说，随着recall的变大，FPR
的大小越小越好。所以我们希望找到的最有点，其实是Recall和FPR差距最大的点。这个点，又叫做约登指数。

```python
maxindex = (recall - FPR).tolist().index(max(recall - FPR))

# 把上述代码放入这段代码中：
plt.figure()
plt.plot(FPR, recall, color='red',
         label='ROC curve (area = %0.2f)' % area)
plt.plot([0, 1], [0, 1], color='black', linestyle='--')

# 我们可以在图像上来看看这个点在哪里
plt.scatter(FPR[maxindex],recall[maxindex],c="black",s=30)

plt.xlim([-0.05, 1.05])
plt.ylim([-0.05, 1.05])
plt.xlabel('False Positive Rate')
plt.ylabel('Recall')
plt.title('Receiver operating characteristic example')
plt.legend(loc="lower right")
plt.show()
```

最佳阈值就这样选取出来了，由于现在我们是使用decision_function来画ROC曲线，所以我们选择出来的最佳阈值其实是最佳距离。如果我们使用的是概率，我们选取的最佳阈值就会使一个概率值了。只要我们让这个距离/概率以上的点，都为正类，让这个距离/概率以下的点都为负类，模型就是最好的：即能够捕捉出少数类，又能够尽量不误伤多数类，整体的精确性和对少数类的捕捉都得到了保证。

# XGBoost

参数

剪枝上的调参顺序是：n_estimators与eta共同调节，gamma或者max_depth，采样和抽样参数（纵向抽样影响更大），最后才是正则化的两个参数

```python
n_estimators       # 多少颗树 不宜过多
subsample          # 随机抽样的时候抽取的样本比例，范围(0,1]
eta/learning_rate  # 集成中的学习率，又称为步长 以控制迭代速率，常用于防止过拟合 默认0.3/0.1 取值范围[0,1]
'''
使用哪种弱评估器。可以输入gbtree，gblinear或dart。
gbtree代表梯度提升树，dart是Dropouts meet Multiple Additive Regression Trees，可译为抛弃提升树，在建树的过
程中会抛弃一部分树，比梯度提升树有更好的防过拟合功能。输入gblinear使用线性模型。
'''
xgb_model/booster  # 默认reg:linear

'''
在实际应用中，正则化参数往往不是我们调参的最优选择，如果真的希望控制模型复杂度，我们会调整gamma而不是调整这两个正则化参数
'''
# 防止过拟合参数  最大深度和gamma只调整一个就好了
L1/L2              # 正则项
alpha              # L1正则项的参数alpha，默认0，取值范围[0, +∞] reg_alpha，默认0，取值范围[0, +∞]
lambda             # L2正则项的参数lambda，默认1，取值范围[0, +∞] reg_lambda，默认1，取值范围[0, +∞]
gamma              # 降低训练集上的表现防止过拟合 默认0，取值范围[0, +∞]
max_depth          # 树的最大深度 默认6
colsample_bytree   # 每次生成树时随机抽样特征的比例 默认1
colsample_bylevel  # 每次生成树的一层时 随机抽样特征的比例 默认1
min_child_weight   # 即叶子节点上的二阶导数之和 类似于样本权重 默认1
```

通常，我们不调整eta，即便调整，一般它也会在[0.01,0.2]之间变动。如果我们希望模型的效果更好，更多的可能是从树本身的角度来说，对树进行剪枝，而不会寄希望于调整。

当alpha和lambda越大，惩罚越重，正则项所占的比例就越大，在尽全力最小化目标函数的最优化方向下，叶子节点数量就会被压制，模型的复杂度就越来越低，所以对于天生过拟合的XGB来说，正则化可以一定程度上提升模型效果

学习曲线的绘制

```python
from sklearn.model_selection import KFold
from xgboost import XGBRegressor as XGBR
from sklearn.model_selection import learning_curve
import matplotlib.pyplot as plt

def plot_learning_curve(estimator,title, X, y, 
                        ax=None, #选择子图
                        ylim=None, #设置纵坐标的取值范围
                        cv=None, #交叉验证
                        n_jobs=None #设定索要使用的线程
                       ):
    
    from sklearn.model_selection import learning_curve
    import matplotlib.pyplot as plt
    import numpy as np
    
    train_sizes, train_scores, test_scores = learning_curve(estimator, X, y
                                                            ,shuffle=True
                                                            ,cv=cv
                                                            ,random_state=420
                                                            ,n_jobs=n_jobs)      
    if ax == None:
        ax = plt.gca()
    else:
        ax = plt.figure()
    ax.set_title(title)
    if ylim is not None:
        ax.set_ylim(*ylim)
    ax.set_xlabel("Training examples")
    ax.set_ylabel("Score")
    ax.grid() #绘制网格，不是必须
    ax.plot(train_sizes, np.mean(train_scores, axis=1), 'o-'
            , color="r",label="Training score")
    ax.plot(train_sizes, np.mean(test_scores, axis=1), 'o-'
            , color="g",label="Test score")
    ax.legend(loc="best")
    return ax

cv = KFold(n_splits=5, shuffle = True, random_state=42) #交叉验证模式

plot_learning_curve(XGBR(n_estimators=100,random_state=420)
                    ,"XGB",Xtrain,Ytrain,ax=None,cv=cv)
plt.show()
```

## n_estimators的调优（subsample调优同理）

300以下较好 除非数据量很大

不考虑方差和偏差的情况下 最好的n_estimators为660 然而样本只有550个

```python
axisx = range(10,1010,50)
rs = []
for i in axisx:
    reg = XGBR(n_estimators=i,random_state=420)
    rs.append(CVS(reg,Xtrain,Ytrain,cv=cv).mean())
print(axisx[rs.index(max(rs))],max(rs))  # 660 0.8046775284172915
plt.figure(figsize=(20,5))
plt.plot(axisx,rs,c="red",label="XGB")
plt.legend()
plt.show()
```

观察最大R2时方差和偏差 最小方差时R2和偏差 最小偏差时R2和方差 它们对应的树

最终细化最小偏差时的数

```python
axisx = range(50,1050,50)
rs = []
var = []
ge = []
for i in axisx:
    reg = XGBR(n_estimators=i,random_state=420)
    cvresult = CVS(reg,Xtrain,Ytrain,cv=cv)
    #记录1-偏差
    rs.append(cvresult.mean())
    #记录方差
    var.append(cvresult.var())
    #计算泛化误差的可控部分
    ge.append((1 - cvresult.mean())**2+cvresult.var())
#打印R2最高所对应的参数取值，并打印这个参数下的方差
print(axisx[rs.index(max(rs))],max(rs),var[rs.index(max(rs))])  # 650 0.80476 0.01053
#打印方差最低时对应的参数取值，并打印这个参数下的R2
print(axisx[var.index(min(var))],rs[var.index(min(var))],min(var))  # 50 0.78577 0.00907
#打印泛化误差可控部分的参数取值，并打印这个参数下的R2，方差以及泛化误差的可控部分
print(axisx[ge.index(min(ge))],rs[ge.index(min(ge))],var[ge.index(min(ge))],min(ge))  # 150 0.80328 0.00974 0.04844
plt.figure(figsize=(20,5))
plt.plot(axisx,rs,c="red",label="XGB")
plt.legend()
plt.show()
```

## xgb实现法

| 输入            | 选用的损失函数                                         |
| --------------- | ------------------------------------------------------ |
| reg:linear      | 使用线性回归的损失函数，均方误差，回归时使用           |
| binary:logistic | 使用逻辑回归的损失函数，对数损失log_loss，二分类时使用 |
| binary:hinge    | 使用支持向量机的损失函数，Hinge Loss，二分类时使用     |
| multi:softmax   | 使用softmax损失函数，多分类时使用                      |

```python
import xgboost as xgb
import pandas as pd
from sklearn.metrics import r2_score

# 使用类DMatrix读取数据
dtrain = xgb.DMatrix(Xtrain,Ytrain)  # 特征矩阵和标签都进行一个传入
dtest = xgb.DMatrix(Xtest,Ytest)

# 写明参数
param = {'silent':True  # 默认为False，通常要手动把它关闭掉
         ,'objective':'reg:linear'
         ,"eta":0.1}
num_round = 180  # n_estimators

# 类train，可以直接导入的参数是训练数据，树的数量，其他参数都需要通过params来导入
bst = xgb.train(param, dtrain, num_round)

# 接口predict
preds = bst.predict(dtest)

r2_score(Ytest,preds)  # 0.9260984298390122
MSE(Ytest,preds)       # 6.87682821415069
```

### xgb的cv

| 指标     | 含义                           |
| -------- | ------------------------------ |
| rmse     | 回归用，调整后的均方误差       |
| mae      | 回归用，绝对平均误差           |
| logloss  | 二分类用，对数损失             |
| mlogloss | 多分类用，对数损失             |
| error    | 分类用，分类误差，等于1-准确率 |
| auc      | 分类用，AUC面积                |

```python
import xgboost as xgb

# 为了便捷，使用全数据
dfull = xgb.DMatrix(X,y)
# 设定参数
param1 = {'silent':True,'obj':'reg:linear',"gamma":0,"eval_metric":"mae"}
num_round = 100
n_fold=5  # sklearn - KFold

cvresult1 = xgb.cv(param1, dfull, num_round,n_fold)

plt.figure(figsize=(20,5))
plt.grid()
plt.plot(range(1,101),cvresult1.iloc[:,0],c="red",label="train,gamma=0")
plt.plot(range(1,101),cvresult1.iloc[:,2],c="orange",label="test,gamma=0")
plt.legend()
```

