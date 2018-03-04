
**TC_shop_location商场中精确定位用户所在商铺比赛**——[比赛官网](https://tianchi.aliyun.com/competition/introduction.htm?spm=5176.100150.711.5.16422009ClSUJf&raceId=231620)

比赛简介：
- 参赛者需要对我们提供的2017年8月份（复赛为2017年7-8月）数据进行店铺、用户、WIFI等各个维度进行数据挖掘和特征创建，并自行创建训练数据中的负样本，进行合适的机器学习训练。在我们提供2017年9月份的数据中，根据当时用户所处的位置和WIFI等环境信息，通过您的算法或模型准确的判断出他当前所在的店铺。

主要工作：
- 第一赛季中，把该问题作为多分类问题对待，并对每个商场建立一个模型；然后进行特征提取，并对其中一些特征做特征变换；最后建立xgboost模型和随机森林模型，进行线性加权，得到最后结果。
- 第二赛季中，依然将问题作为多分类问题看待，由于线上数据量过大，能量包限制等问题，所以只把每条交易记录Wifi强度前十的WifiID和WiFi强度作为主要特征，采用MaxCompute中的PS-SMART模型，进行参数调优。

最终成绩，初赛92/2845，复赛32/2845。


----------

比赛详细工作及各时间段工作如下：
#### 初赛
- 这个比赛是我第一次使用机器学习算法解决天池比赛上的问题，一开始也摸不着北，不知道如何解决，或者说不知道该把这个问题作为什么类型的问题来建模，看了官方给的说明以及群里的讨论，有说作为多分类，有说作为二分类，有说作为定位问题，有说作为推荐问题等等，当时也是一筹莫展。
- 因为题目的背景是在室内环境下，GPS定位精度不高，想要根据手机的WIFI信息来对用户定位，于是我们搜索了一些wifi定位的论文和博客（[室内定位系列(1)—WiFi位置指纹](http://www.cnblogs.com/rubbninja/p/6120964.html)，[基于Wi-Fi的室内定位在美团总部的实践和应用（上）](https://tech.meituan.com/mt-wifi-locate-practice-part1.html)等）以及题目所给出的数据，我们使用了一个规则来求解问题，即使用test某条记录和train中记录**WiFi强度差的平方求均值**来计算test中某条记录和train中哪条记录最接近，即将该条记录的shop_id作为test该条记录的shop_id。这种方法求解出来的答案准确率大概在85%，这在赛季一开始名次还比较高。也从侧面说明了这个问题中WiFi强度是一个非常强的特征。
- 大概过了一周左右，好像大家都找到了用机器学习方法求解的套路，分数也都达到了90%，我们也在想如何使用机器学习方法来求解该问题。主要遇到的麻烦是，主要特征wifi_infos是如下的字段，里面包含了10个wifi_id,wifi_strength,是否连接。(`一般是10个，也有不足10个的情况`) 训练集每条记录都包含了这样的wifi_infos，涉及到的wifi_id非常多，我们不知道如何将wifi_infos转换为特征，如果使用Onehot编码，那么特征值将会非常多，成千上万个特征![Alt text](./1519873917757.png)
- 过后思考再三，我们按照97个商场划分训练集，对于每个商场都构建了一个多分类模型，将wifi_infos按照wifi_id拆开，同商场内的每个wifi_id作为一个特征，将每条记录对应wifi_id的强度作为特征值，得到是一个非常稀疏的矩阵，后面我们做了一个改善即不选取商场内的所有wifi_id作为特征，而是按照wifi_id出现次数排序，选取前30%的wifi_id（另外一个队友选取的是wifi次数大于2的wifi），这样得到训练集维度大概在2000维左右。
- 后续还加入的有效特征：1、时间特征，如日期小时，是否周末；2、用户的最强wifi强度，用户有无连接以及连接时候的wifi强度；3、经纬度特征；
- 分别使用了xgboost、lightgbm、random forest、NN等模型，后来使用了xgboost和random forest进行线性加权融合，得到了初赛最好成绩——91.76%，初赛排名92/2845。

初赛思考：初赛时，我们团队本身的想法以及比赛群里的讨论，发现把问题当做多分类来求解只能达到一个还不错的成绩，但是准确率想再往上提高，基本不可能了，因为现在多分类求解将wifi_infos拆开，已经不能再围绕wifi来增加特征了，所以分数到达一个瓶颈，从群里讨论来看，前几名是用的是二分类模型。队伍中有一名同学也是尝试二分类，但是候选集的规则不太好，初赛二分类最后只有88%左右。融合方法尝试过线性加权，Stack等，最终还是xgboost与random forest按6:4加权得到最好成绩。

初赛相关数据：链接:https://pan.baidu.com/s/1jJqOkVO  密码:zmhm

#### 复赛
复赛依然沿用的是第一赛季的思路，只是复赛使用的是阿里的MaxCompute平台，怎么说呢，感觉作比赛的都是给人试平台找bug的，阿里的MaxCompute平台主要分为，数据预处理（好像叫做PAI）和机器学习平台，数据预处理可以在线下使用《阿里云odpsSql手册.pdf》中定义的UDF和UDTF函数，将Java程序达成jar包，上传到MaxCompute平台对数据做处理，也可以直接使用手册中的SQL语句在MaxCompute中对数据预处理。 
数据预处理后在机器学习平台中选择合适的模型，图形化的对数据训练和预测。


----------
