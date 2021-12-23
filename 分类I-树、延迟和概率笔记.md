# 准备训练和测试数据集
一上来就发现，数据集找不到，搜索一番，终于在另外一个包找到了数据集。
```
# install.packages("C50")
# library(C50)
# data('churn', package = 'C50')
# install.packages("modeldata")
# https://stackoverflow.com/questions/60506936/data-set-churn-not-found-in-package-c50
library(modeldata)
data(mlc_churn)
churn <- mlc_churn
# 7:3分训练和测试集
set.seed(2)
ind <- sample(2,nrow(churnTrain),replace = TRUE,
              prob = c(0.7,0.3))
trainset <- churnTrain[ind==1,]
testset <- churnTrain[ind==2,]
```
这个数据集和书中的略有区别，不过应该是包含的关系，这个数据的样本更多，应该不影响的。
**扩展：split函数完成训练和测试的划分**
```
split.data <- function(data, p= 0.7, s= 666){
  set.seed(s)
  index <- sample(1:dim(data)[1])
  train <- data[index[1:floor(dim(data)[1]*p)],]
  test <-  data[index[((ceiling(dim(data)[1]*p))+1):dim(data)[1]],]
  return(list(train=train,test=test))
}

li <- split.data(churnTrain)
```
# 使用递归分割树建立分类模型
递归和分割是这个算法的两个步骤。ＣＰ是成本复杂度参数．决策树算法的不足是容易产生偏差和过度适应问题，条件推理树可以克服偏差，过度适应可以借助随机森林方法或树的修剪来解决。
```
library(rpart)
churn.rp <- rpart(churn~., data=trainset)
plotcp(churn.rp)
summary(churn.rp)
```
![](https://upload-images.jianshu.io/upload_images/6644753-e7931be1ca720f1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 5.4 递归分割树可视化
plot和text函数绘制分类树。
```
plot(churn.rp, margin = 0.1) # 边框
text(churn.rp, all = TRUE, use.n = TRUE) # use.n每个类别实际观测个数
# 改变参数来调整显示结果
plot(churn.rp, uniform = TRUE, branch = 0.6, margin = 0.1) # brach 设置shoudler
text(churn.rp,all = TRUE, use.n = TRUE)
```
![](https://upload-images.jianshu.io/upload_images/6644753-a3b629003063ef87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![参数只改变了树的形状？字好像还不好看](https://upload-images.jianshu.io/upload_images/6644753-3edbea07eee2a9a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 5.5 评测递归分割树的分类能力
```
# 预测
predictions <- predict(churn.rp, testset, type = "class")
table(testset$churn, predictions)
# #############
     predictions
       yes   no
  yes  133   81
  no    29 1278
# 生成混淆矩阵
library(caret)
confusionMatrix(table(predictions, testset$churn))
Confusion Matrix and Statistics
# ##############
           
predictions  yes   no
        yes  133   29
        no    81 1278
                                          
               Accuracy : 0.9277          
                 95% CI : (0.9135, 0.9402)
    No Information Rate : 0.8593          
    P-Value [Acc > NIR] : < 2.2e-16       
                                          
                  Kappa : 0.6671   
```
# 5.6 递归分割树剪枝
有时，需要修剪分类描述能力较弱的规则，以避免过度适应，并提高预测正确率，这里使用成本复杂度方法。
```
min(churn.rp$cptable[,"xerror"])
[1] 0.4523327
which.min(churn.rp$cptable[,"xerror"])
5 
5 
# 最小成本复杂度参数
churn.cp <- churn.rp$cptable[5,"CP"]
churn.cp 
[1] 0.01014199
# 修剪
prune.tree <- prune(churn.rp, cp=churn.cp)
plot(churn.rp, margin = 0.1)
text(churn.rp,all = TRUE, use.n = TRUE)
# 混淆矩阵，略低于修剪前，避免过拟合
confusionMatrix(table(predictions, testset$churn))
# ################
```
![好像没怎么发现哪修剪了](https://upload-images.jianshu.io/upload_images/6644753-e1a7b52806b4b1d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 5.7 使用条件推理树建立分类模型
rpart传统决策树算法之外，条件推理树ctree是另外一类比较常用的基于树的分类算法。同样对非独立变量来实现对数据的递归划分处理。不同在于，条件推理树选择分裂变量的依据是显著性测量的结果，而不是信息最大化方法，rpart里使用了基尼系数，这个不是表征贫富差距的。
```
# 条件推理树
library(party)
ctree.moddel <- ctree(churn~., data = trainset)
ctree.moddel
```
# 5.8 条件推理树可视化
```
# 可视化
plot(ctree.moddel)
daycharhe.model <- ctree(churn~total_day_charge, data = trainset)
plot(daycharhe.model)
```
![稍微觉得这树好看点](https://upload-images.jianshu.io/upload_images/6644753-e0f08f76a87b7e69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**简化一下**
![简化太多啦](https://upload-images.jianshu.io/upload_images/6644753-1e074e9b3d953463.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 5.9 评估预测能力
```
 # 预测
> ctree.predict<- predict(ctree.moddel, testset)
> table(ctree.predict, testset$churn)
             
ctree.predict  yes   no
          yes  139    9
          no    75 1298
confusionMatrix(table(ctree.predict, testset$churn))
# #######################
Confusion Matrix and Statistics

             
ctree.predict  yes   no
          yes  139    9
          no    75 1298
                                          
               Accuracy : 0.9448          
                 95% CI : (0.9321, 0.9557)
    No Information Rate : 0.8593          
    P-Value [Acc > NIR] : < 2.2e-16  
# 概率
> treeresponse(ctree.moddel, newdata = testset[1:5,])
[[1]]
[1] 0.02715356 0.97284644

[[2]]
[1] 0.06842105 0.93157895

[[3]]
[1] 0.06842105 0.93157895

[[4]]
[1] 0.06842105 0.93157895

[[5]]
[1] 0.02715356 0.97284644
```
# 5.10 使用k临接分类算法
是一种无参惰性学习方法，不会对数据分布做任何假设，不要求算法具备显性学习过程。
```
install.packages("class")
library(class)
levels(trainset$international_plan) = list("0" = "no", "1" = "yes")
levels(trainset$voice_mail_plan) = list("0" = "no", "1" = "yes")
levels(testset$international_plan) = list("0" = "no", "1" = "yes")
levels(testset$voice_mail_plan) = list("0" = "no", "1" = "yes")

churn.knn <- knn(trainset[,!names(trainset) %in% c("churn", "area_code", "state" )],
                 testset[,!names(testset)  %in% c("churn", "area_code", "state" )], trainset$churn, k=3)
summary(churn.knn)
plot(churn.knn)
library(caret)
confusionMatrix(table(testset$churn,churn.knn))
# ########################
Confusion Matrix and Statistics

     churn.knn
       yes   no
  yes   76  138
  no    46 1261
                                         
               Accuracy : 0.879          
                 95% CI : (0.8616, 0.895)
    No Information Rate : 0.9198         
    P-Value [Acc > NIR] : 1              
                                         
                  Kappa : 0.3901  
```
knn算法采用相似性距离来训练和分类，比如使用欧氏距离或曼哈顿距离，k=1，会分配样本到距离其最近的类别，如果k较小，可能过拟合，过大，低拟合，可以交叉检验获得一个合适值。
优势在于学习成本为0，不需要假设分布，可以处理任意类型数据；不足在于难以理解，数据集较大计算代价非常高，高维数据要先降维。字符类型数据要先处理成整型，k=3分配到最近3个簇中。kknn包可以提供带权重的k邻近算法、回归和聚类。
# 5.11 使用逻辑回归
属于基于概率统计的算法，logit函数可以执行，glm family指定为binomial也是逻辑回归算法。
```
# logistic
fit <- glm(churn~., data = trainset,family = binomial)
summary(fit)
# 去除非显著变量

fit <- glm(churn~international_plan + voice_mail_plan + total_intl_calls + number_customer_service_calls , data = trainset,family = binomial)
summary(fit)
pred <- predict(fit, testset, type = "response")
pred <- predict(fit, testset, type = "response")
Class <- pred > .5
summary(Class)
   Mode   FALSE    TRUE 
logical      44    1477 
pred_class <- churn.mod
pred_class[pred <=.5] = 1- pred_class[pred<=.5]
ctb <- table(churn.mod, pred_class)
ctb
# ###########
         pred_class
churn.mod    0    1
        0 1287   20
        1   24  190
 confusionMatrix(ctb)
Confusion Matrix and Statistics

         pred_class
churn.mod    0    1
        0 1287   20
        1   24  190
                                          
               Accuracy : 0.9711          
                 95% CI : (0.9614, 0.9789)
    No Information Rate : 0.8619          
    P-Value [Acc > NIR] : <2e-16          
                                          
                  Kappa : 0.8794 
```
逻辑回归易于理解，直接输出概率和置信区间，能迅速合并新的数据集，更新分类模型。不足在于无法处理多重共线性总是，解释变量必须线性无关。
# 5.12 使用朴素贝叶斯分类算法
也是基于概率的分类器，假设样本属性之间相互独立。
```
library(e1071)
classifer <- naiveBayes(trainset[,!names(trainset) %in% c("churn")], trainset$churn)
classifer
# ##############
Naive Bayes Classifier for Discrete Predictors

Call:
naiveBayes.default(x = trainset[, !names(trainset) %in% c("churn")], 
    y = trainset$churn)
# ###############
A-priori probabilities:
trainset$churn
      yes        no 
0.1417074 0.8582926 
bayes.table <- table(predict(classifer, testset[,!names(trainset) %in% c("churn")]), testset$churn)
bayes.table
# #############     
       yes   no
  yes  104   52
  no   110 1255
library(caret)
confusionMatrix(bayes.table)
Confusion Matrix and Statistics
# ############
     
       yes   no
  yes  104   52
  no   110 1255
                                          
               Accuracy : 0.8935          
                 95% CI : (0.8769, 0.9086)
    No Information Rate : 0.8593          
    P-Value [Acc > NIR] : 4.220e-05       
                                          
                  Kappa : 0.5032  
```
分类评估基于就是上面这样一个套路了。朴素由叶斯算法假设特征变量都是条件独立的，优势相对简单，应用直接，适合训练数据集规模树比较小，可能存在缺失或者数据噪音的情况。不足在于上面的条件相互独立和同等重要，在实际世界中很难实现。
总结本章，见如下图片：
![](https://upload-images.jianshu.io/upload_images/6644753-4562dd390069df69.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


