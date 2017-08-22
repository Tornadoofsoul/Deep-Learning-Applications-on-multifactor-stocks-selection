# Deep-Learning-Applications-on-multifactor-stocks-selection
一、	基于深度学习的神经网络多因子选股模型策略思想

我们根据财务、技术、成长、价值、质量等多方面因素构建了多因子选股模型。传统的多因子模型在构建大类因子特征时往往依赖于投资者的主观判断和逻辑推理。机器学习等量化模型，依据某种机制，构建出一个具有自适应和自动学习特征的模型，被广泛应用于多因子模型的构建。相较于SVM、Logistic Regression等机器学习分类器，深度学习可以通过构建具有很多隐层的机器学习模型和海量的训练数据，来学习到更有用的因子特征。因而“深度模型”只是手段，我们的最终目标是利用深度神经网络进行多因子的“特征学习”，从而最终提升分类或预测的准确性。

该策略目前，基于现有44维因子，通过神经网络模型学习因子特征，从而选出市场表现最优的100只股票，构建最优投资组合。
二、主要模型策略
1.	44个因子的autoencoder选股模型
1）目标：
AutoEncoder自动编码器就是一种尽可能复现输入信号的神经网络。为了实现这种复现，自动编码器就必须捕捉可以代表输入数据的最重要的因素，就像PCA那样，找到可以代表原信息的主要成分。我们通过AutoEncoder这种对44维因子信息进行压缩至8维因子，再在8维因子上构建分类器，输出股票分类，选出排名前100的只最优股票。 

2）流程：
•	构建多层自动编码器，输入历史上44个股票因子打分，通过逐层训练神经网络，提取因子主要特征。
•	划分标签，构建训练、测试集，详见程序说明。 
•	分类：将最后层提取的特征code输入到最后的softmax分类器，通过有标签样本，训练这个分类器。
•	输出股票分类和打分。

3）进度/结果：
结果：预测效果并不好，由Autoencoder处理得到的降维后8维因子为无监督训练的结果，并不能对股票收益率起到很好的预测效果。最后放弃了该策略的研究，转而研究DNN模型。
详细代码见project/ autoencoder+multifactor文件夹，autoencoder+softmax.py文件


2.	44个因子的Deep Neural Network 选股模型
1）	目标：
不再对44维因子进行压缩，而是直接全部输入分类器，将Deep Neural Network整张网络作为一个分类器，输出股票分类和打分
2）流程：具体操作见程序说明

•	确定滚动回测周期，三年为训练神经网络周期，调仓周期为20天。构建对应的因子训练样本集和测试集（详见程序说明，DNN-classificiation-XXfactor.ipynb）
 
•	设定神经网络结构，训练网络，参数调优
•	输出股票分类、打分
•	对股票打分进行因子检验
•	将股票打分输入回测模板，输出每个换仓日预测表现最优的100只股票，根据历史行情计算收益水平、统计portfolio表现、计算基金净值。

3）进度：全部完成
4）结果：由44维因子和Deep Neural Network选出的100只股票历史表现未战胜六大类等权策略。详见performance文件夹的回测报告。


3.	21个因子的 Deep Neural Network 选股模型
目标：将44维因子中与收益率相关度不高的因子剔除，剩下21维因子，重复44因子DNN模型所有步骤，查看回测结果。
流程：同44维因子
进度：全部完成
结果：21个因子+DNN模型的回测结果较44维因子略有提升，仍未战胜六大类等权。
 

4.	基于技术因子的 Deep Neural Network 选股模型（中长期因子-短期因子分开考虑）
1）	目标：上述几个模型的表现效果并不令人满意。我们认为将中长期因子和短期因子混合在一起作为模型输入，会导致因子间干扰增多，模型预测效果下降。因此，我们先用中长期因子模型过滤筛选出表现较强的500只，最后只考虑技术因子，用Deep Neural Network做分类器对股票进行分类预测，选出表现最优的100只股票。

2）流程：
•	确定滚动回测周期，三年为训练神经网络周期，调仓周期为10天。构建对应的因子训练样本集和测试集（详见程序说明，DNN-classificiation-XXfactor.ipynb部分）

正式回测时，三年期训练数据表现并不好。技术因子是短期因子，我们应该缩短训练神经网络的周期，尝试用一年、一年半、两年等更短期限做训练样本集。

•	设定神经网络结构，训练网络，参数调优。
•	输出股票分类、打分
•	对股票打分进行因子检验
•	将股票打分输入回测模板，输出每个换仓日预测表现最优的100只股票，根据历史行情计算收益水平、统计portfolio表现、计算基金净值。

3）进度：未做因子检验，投资组合回测。
4）结果： 
对由技术因子筛选出的100只最优股票进行了初步回测，发现相同时期内，基金净值累计为2.5，尚未超过44/21因子模型。

5）总结失败原因：
（1）	还需要对回测周期进行调整，或许能提升模型效果
（2）	神经网络未学习出技术因子与股票收益率间的非线性关系，导致模型预测效果非常差。需要调整DNN模型结构。
（3）	现有技术因子有效的太少，继续扩充技术因子数据库
（4）	不应该用全市场的技术因子数据训练模型。训练集应该只由500只股票的技术因子构成。
可根据以上几点继续对该模型进行改进。

三、程序说明
1.GetDataXXfactor.py 
从数据库中下载因子并按交易日期整理成csv文件的程序。可获得属于六大类因子的所有44个因子，和股票收益率又显著相关性的21个因子，与股票收益率成非线性关系的技术（tech）因子。所有csv文件详见上传至钉盘的data文件夹，每一个因子又一个文件夹，里面每一个csv文件是一个交易日内全市场股票的全部因子数据。

GetDataTechFactor.py
中的一步程序还会下载数据库中，由中长期因子选择出的最优500只股票。作为技术因子的预测股票池。
2.Processdata-XXfactor.py
合成神经网络所需要的训练train数据集，只提取每个因子的z-score数据，将所有日期、所有股票的因子数据合成一张大表做训练数据集（日期-股票*44/21/7维因子）。存储数据至Train.csv.上传至terminal。
#操作：逐个打开每个因子的文件夹，逐个打开每个时间段的数据csv，提取z-score，将后一天的数据衔接在前一天后。遍历完一个因子文件夹后，另起一列append另一个因子整个时间段的z-score
3.Get20dayReturn.py
在对44/21个因子构建训练集标签时，以20天为换仓周期，分别计算20天交易日期间收益率（现成数据，未对冲中证800指标），并划分标签。
函数Get_2label(Retu,perc=0.3)
#计算标签，label分为两档，20天换仓期间收益率表现为前30%的为强势股标为1，后30%为弱势标为0。
函数 Get_5label(Ret):  
将股票分为五档 0%-20%:0, 20%-40%:1,40%-60%:2,60%-80%:3,80%-100%:4最强。
4.Get10dayReturn.py，在对技术因子构建标签时，首先下载10天间隔交易日除权复权后价格，并计算10天交易日期间收益率（由adjclose计算）。最后对冲中证800收益率计算alpha收益。
计算label：标签根据alpha收益进行划分。10天换仓期间alpha收益率表现为前50%，且alpha大于0的为强势股标为1，后50%或alpha小于0的为弱势股票标为0。
5.DNN classification-44/21/TECH.ipynb
安装包：keras、tensorflow、 theano。
神经网络的所有相关计算过程均有keras软件包完成。
步骤：
a.导入全历史的因子数据和收益率标签。
b.构建滚动回测窗口，确定训练神经网络的窗口期和换仓周期，构建训练集和测试集。
1）	21/44因子在训练神经网络时，均以三年期全市场因子和标签数据（由收益率算得）作为训练数据集有监督训练整个神经网络。20天为换仓周期，由换仓日当天的全市场股票因子数据作为测试数据集，预测下一期股票分类。

训练数据集构成：三年期全市场21/44个因子数据 + 对应标收益率签数据。
测试数据集构成：将换仓当日全市场股票20天收益率+换仓日当天全市场21/44维因子数据合成测试数据集，为无标签数据。

2）	技术因子在训练神经网络时，均以三年期全市场的7个技术因子和标签数据（由alpha算得）作为训练数据集有监督训练整个神经网络。在构建测试数据集时，1 0天为换仓周期，先通过中长期大类因子过滤出的最强500支股票，换仓当日仅由500只股票的因子数据和alpha收益作为测试数据集，预测下一期股票分类。

训练数据集构成：三年期全市场股票的7个技术因子+对应alpha标签数据。
测试数据集构成：换仓日由中长期因子过滤而得的500只优势股票的10天期间alpha收益+的7维技术因子，为无标签数据。

注: 正式回测时，三年期训练数据表现并不好。鉴于技术因子是短期因子，我们应该缩短训练神经网络的周期，尝试用一年、一年半、两年等更短期限做训练样本集。

3）	其中，标签数据TrainY为了适应keras模板格式，需用utils.to_categorical函数转化成向量格式。

c.Deep Neural Network 参数调整（目前最优）
层数 dim：目前最优为4层，前三层训练神经网络，最后一层接softmax分类器
Class个数：nb_cl
每层网络节点个数：目前最优为100个
输入（因子维度）——第一层100个节点（Dropout：25%）——第二层100个节点（Dropout：25%）——第三层100个节点（Dropout：25%）——输出(softmax:两层分类)
激活函数：relu, loss下降最快
d.DNN模型训练、预测
1)训练
Optimizer: ‘adadelta’
Loss: ‘categorical_crossentropy’  
训练次数：nb_epoch=100
Batch_size=1000, 每次训练更新的样本集中数据个数
Validation_split: validation数据长度 20%
问题：early_stopping=EarlyStopping(monitor='val_loss', patience=2)
为了防止过拟合，设置在validation set loss不再下降时停止训练，模型尚未收敛，输出分类结果不稳定。

函数neural_network:设定神经网络架构
Model.fit训练神经网络

2）预测
Model.predict_classes 输出预测分类
Model.predict_proba 输出预测分类概率
将预测分类为1的概率作为股票打分proba1，记录到long_cnn_day.csv文件。

3)	初步收益统计
以做多前100只股票，做空后100只股票作为策略，统计每次换仓收益，累计收益水平。

6.因子检验：21/44/tech factor test_DNN.ipynb
因子检验回测模板。输入股票的预测为分类1的概率做因子，输出全市场股票n组分档。
结果

以44 factor test为例，testIC可达0.1064.
ic
count  56.000000
mean    0.106405
std     0.133368
min    -0.353180
25%     0.038587
50%     0.102202
75%     0.188705
max     0.328205


 

7.最优100只股票收益回测： backtest_XXfactor.
回测模板，输出每个换仓日预测表现最优的100只股票，根据历史行情计算收益水平、统计portfolio表现、计算基金净值。
21个因子、44个因子神经网络选股模型均未战胜六大类等权策略。21因子略优于44因子。
21、44个因子回测报告可见performance文件夹。
Tech因子回测效果非常差，暂未带入模板生成报告。

Entire data start date: 2011-01-17
Entire data end date: 2015-08-26

21个因子裸多
Backtest Months: 53
Performance statistics	Backtest
annual_return	0.37
annual_volatility	0.29
sharpe_ratio	1.22
calmar_ratio	0.75
stability_of_timeseries	0.83
max_drawdown	-0.49
omega_ratio	1.24
sortino_ratio	1.67
skew	-0.84
kurtosis	3.75
tail_ratio	0.90
common_sense_ratio	1.23
information_ratio	0.17
alpha	0.25
beta	1.00
Worst Drawdown Periods	net drawdown in %	peak date	valley date	recovery date	duration
0	49.33	2015-06-12	2015-07-08	NaT	NaN
1	35.25	2011-03-25	2012-01-05	2013-03-06	509
2	16.83	2013-05-30	2013-07-08	2013-08-29	66
3	11.00	2014-12-04	2014-12-23	2015-01-22	36
4	8.33	2013-11-29	2014-01-20	2014-02-14	56
 


21因子对冲
Backtest Months: 53
Performance statistics	Backtest
annual_return	0.32
annual_volatility	0.09
sharpe_ratio	3.11
calmar_ratio	2.91
stability_of_timeseries	0.97
max_drawdown	-0.11
omega_ratio	1.75
sortino_ratio	5.24
skew	0.22
kurtosis	7.70
tail_ratio	1.34
common_sense_ratio	1.76
information_ratio	0.04
alpha	0.28
beta	-0.02
Worst Drawdown Periods	net drawdown in %	peak date	valley date	recovery date	duration
0	10.87	2014-11-07	2015-01-05	2015-04-03	106
1	9.34	2015-06-26	2015-07-08	2015-08-05	29
2	5.50	2012-06-20	2012-07-16	2012-08-16	42
3	5.46	2013-05-23	2013-07-24	2013-08-16	62
4	5.08	2013-12-17	2014-02-10	2014-03-14	64

 
21个因子裸多与六大类等全对比，灰色线六大类，绿色线神经网络。
 










