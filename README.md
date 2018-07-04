环境配置：

    Anaconda 4.4.10, python 3.6.4

主要依赖库：

    pandas 0.22.0, numpy 1.14.2, scikit-learn 0.19.1, lightgbm 2.2.1

步骤说明：

    按文件名顺序依次运行代码，先后完成预处理、特征工程、模型训练、测试集预测与模型结果融合
    
预处理：
    
    预处理有个特别的地方就是对merge完的训练集进行了分块，后续利用这个分块进行转化率特征的提取以及统一利用其中一个分块数据作为验证集。
    
特征提取：

    五大类特征，投放量（click）、投放比例（ratio）、转化率（cvr）、特殊转化率（CV_cvr）、多值长度（length），每类特征基本都做了一维字段和二维组合字段的统计。值得注意的是转化率利用预处理所得的分块标签独立出一个分块验证集不加入统计，其余分块做dropout交叉统计，测试集则用全部训练集数据进行统计。此外，我们发现一些多值字段的重要性很高，所以利用了lightgbm特征重要性对ct\marriage\interest字段的稀疏编码矩阵进行了提取，提取出排名前20的编码特征与其他单值特征进行类似上述cvr的统计生成CV_cvr的统计，这组特征和cvr的效果几乎相当。
    
 特征筛选：
 
    内存原因，没办法一次性加载所有特征，故对每组特征进行组内筛选，对组内特征进行重要性排序，排序完筛选办法有两种，一种是按照排名进行前向搜索，另一种就是直接测试前n*5(n位正整数)个特征的效果，由于多数统计特征之间相关性很高，大概每组30+个特征的信息就能代表整组特征所包含的信息。第一种精度会好一些单速度慢，第二种办法就相对快很多。

模型融合：
    
    由于数据量过大，lgb根据分块数据与分组特征跑了很多个子模型，最后根据验证集的多组预测值进行auc排序后，依次百分比（list(range(0,101))*0.01）遍历加权以获得最佳权值，再将同样的权值应用到测试集的预测结果上，这样每多加权一个子模型，验证集的auc只会大于等于加权这个子模型之前的auc。整个加权过程其实就类似于是一种线性拟合，也可以利用各个子模型的验证集和测试集的预测结果作为特征，利用验证集的标签作为真实标签，采用xgboost等模型进行训练，这样效果与之前的遍历加权差不多。
    
体会不足：

    模型方面参数没有做优化，20%的验证集相对来说选得比较大，初赛数据加得比较晚导致之前筛选出的特征不是加了初赛数据之后得训练集的最优特征。
    
致谢：

    本次参赛特别感谢byran的baseline，让我们学会了使用稀疏矩阵。另外，也特别感谢郭大开源的nffm，没有nffm，单靠lgb我们最后只能达到0.763+的成绩。最后，感谢两位队友（鱼遇雨欲语与余、全靠队友带）的共同努力，儿须成名酒须醉。
