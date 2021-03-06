# 简介
![](https://upload-images.jianshu.io/upload_images/6644753-7516f776f743b2c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这本书是一位韩国的数据科学家写的书，书并不新，几年前的，虚拟机都已经过时，但全书读下来最大的感受是内容写的很详细，可以照着步骤实践，加深了对机器学习的理解，或者说是一本入门好书。当然，仅靠一本书，一次接触一个知识点，我们可能很难掌握一个技能，只有项目实践，如果暂时没有项目，可以用公开数据集，跟着这本书的代码测试下，我会把markdown文件上传至github和gitee，欢迎交流，纠正错误，并给出少部分我在笔记中没重复成功的原因。


## 附上本书目录：
目录
译者序
前言
作者简介
审校者简介
第1章基于R实践机器学习
1.1简介
1.2下载和安装R
1.3下载和安装R Studio
1.4包的安装和加载
1.5数据读写
1.6使用R实现数据操作
1.7应用简单统计
1.8数据可视化
1.9获取用于机器学习的数据集
第2章挖掘RMSTitanic数据集
2.1简介
2.2从CSV文件中读取Titanic数据集
2.3根据数据类型进行转换
2.4检测缺失值
2.5插补缺失值
2.6识别和可视化数据
2.7基于决策树预测获救乘客
2.8基于混淆矩阵验证预测结果的准确性
2.9使用ROC曲线评估性能
第3章R和统计
3.1简介
3.2理解R中的数据采样
3.3在R中控制概率分布
3.4在R中进行一元描述统计
3.5在R中进行多元相关分析
3.6进行多元线性回归分析
3.7执行二项分布检验
3.8执行t检验
3.9执行Kolmogorov—Smirnov检验
3.10理解Wilcoxon秩和检验及Wilcoxon符号秩检验
3.11实施皮尔森卡方检验
3.12进行单因素方差分析
3.13进行双因素方差分析
第4章理解回归分析
4.1简介
4.2调用1m函数构建线性回归模型
4.3输出线性模型的特征信息
4.4使用线性回归模型预测未知值
4.5生成模型的诊断图
4.6利用1m函数生成多项式回归模型
4.7调用rlm函数生成稳健线性回归模型
4.8在SLID数据集上研究线性回归案例
4.9基于高斯模型的广义线性回归
4.10基于泊松模型的广义线性回归
4.11基于二项模型的广义线性回归
4.12利用广义加性模型处理数据
4.13可视化广义加性模型
4.14诊断广义加性模型
第5章分类Ⅰ——树、延迟和概率
5.1简介
5.2准备训练和测试数据集
5.3使用递归分割树建立分类模型
5.4递归分割树可视化
5.5评测递归分割树的预测能力
5.6递归分割树剪枝
5.7使用条件推理树建立分类模型
5.8条件推理树可视化
5.9评测条件推理树的预测能力
5，10使用k近邻分类算法
5.11使用逻辑回归分类算法
5.12使用朴素贝叶斯分类算法
第6章分类Ⅱ——神经网络和SVM
6.1简介
6.2使用支持向量机完成数据分类
6_3选择支持向量机的惩罚因子
6.4实现SVM模型的可视化
6.5基于支持向量机训练模型实现类预测
6.6调整支持向量机
6.7利用neuralnet包训练神经网络模型
6.8可视化由neuralnet包得到的神经网络模型
6.9基于neuralnet包得到的模型实现类标号预测
6.10利用nnet包训练神经网络模型
6.11基于nnet包得到的模型实现类标号预测
第7章模型评估
7.1简介
7.2基于k折交叉验证方法评测模型性能
7.3利用e1071包完成交叉验证
7.4利用caret包完成交叉检验
7.5利用caret包对变量重要程度排序
7.6利用rmlner包对变量重要程度排序
7.7利用caret包找到高度关联的特征
7.8利用caret包选择特征
7.9评测回归模型的性能
7.10利用混淆矩阵评测模型的预测能力
7.11利用ROCR评测模型的预测能力
7.12利用caret包比较ROC曲线
7.13利用caret包比较模型性能差异
第8章集成学习
8.1简介
8.2使用bagging方法对数据分类
8.3基于bagging方法进行交叉验证
8.4使用boosting方法对数据分类
8.5基于boosting方法进行交叉验证
8.6使用gradientboosting方法对数据分类
8.7计算分类器边缘
8.8计算集成分类算法的误差演变
8.9使用随机森林方法对数据分类
8.10估算不同分类器的预测误差
第9章聚类
9.1简介
9.2使用层次聚类处理数据
9.3将树分成簇
9.4使用k均值方法处理数据
9.5绘制二元聚类图
9.6聚类算法比较
9.7从簇中抽取轮廓信息
9.8获得优化的k均值聚类
9.9使用密度聚类方法处理数据
9.10使用基于模型的聚类方法处理数据
9.11相异度矩阵的可视化
9.12使用外部验证评估聚类效果
第10章关联分析和序列挖掘
10.1简介
10.2将数据转换成事务数据
10.3展示事务及关联
10.4使用Apriori规则完成关联挖掘
10.5去掉冗余规则
10.6关联规则的可视化
10.7使用Eclat挖掘频繁项集
10.8生成时态事务数据
10.9使用cSPADE挖掘频繁时序模式
第11章降维
11.1简介
11.2使用FSelector完成特征筛选
11.3使用PCA进行降维
11.4使用scree测试确定主成分数
11.5使用Kaiser方法确定主成分数
11.6使用主成分分析散点图可视化多元变量
11.7使用MDS进行降维
11.8使用SVD进行降维
11.9使用SVD进行图像压缩
11.10使用ISOMAP进行非线性降维
11.11使用局部线性嵌入法进行非线性降维
第12章大数据分析（R和Hadoop）
12.1简介
12.2准备RHadoop环境
12.3安装rmr2
12.4安装rhdfs
12.5在thdfs中操作HDFS
12.6在RHadoop中解决单词计数问题
12.7比较RMapReduce程序和标准R程序的性能差别
12.8测试和调试rmr2程序
12.9安装plymlr
12.10使用plyrmr处理数据
12.11在RHadoop中实施机器学习
12.12在AmazonEMR环境中配置RHadoop机群
附录AR和机器学习的资源
附录BTitanic幸存者的数据集
