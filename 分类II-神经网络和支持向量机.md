支持向量机可以做到全局最优，而神经网络容易陷入多重局部最优。libsvm和SVMLite都是非常流行的支持向量机工具，e1071包提供了libsvm的实现，klap包提供了对后者的实现。
SVM优势在于利用了面向工程问题的核函数，能够提供准确度非常高的模型，同时借助正则项可以避免模型的过度适应，用户不必担心诸如局部最优和多重共线性难题，弊端是训练测试速度慢，模型处理时间冗长，不适合规模庞大数据集。和神经网络一样，都属于黑盒算法，结果较难解释。另外如何确定合适核函数，也是一个难点，正则化也是需要考虑的问题。
gamma函数决定分离超平面的形状，默认为数据维度的倒数，提高它的值通常会增加支持向量的数量。考虑到成本函数，默认值通常为1，此时正则项也是常数，正则项越大，边界越小。
```
library(e1071)
# install.packages("C50")
# library(C50)
# data('churn', package = 'C50')
# install.packages("modeldata")
# https://stackoverflow.com/questions/60506936/data-set-churn-not-found-in-package-c50
library(modeldata)
data(mlc_churn)
churn <- mlc_churn
churnTrain <- churn[,!names(churn) %in% c('state',
                    'area_code', "account_length")]
set.seed(2)
ind <- sample(2,nrow(churnTrain),replace = TRUE,
              prob = c(0.7,0.3))
trainset <- churnTrain[ind==1,]
testset <- churnTrain[ind==2,]
model <- svm(churn~., data = trainset, kernel = "radial", cost = 1,
             gamma=1/ncol(trainset))
summary(model)
Call:
  svm(formula = churn ~ ., data = trainset, kernel = "radial", 
      cost = 1, gamma = 1/ncol(trainset))
# SVMLight
install.packages("klaR")
library(klaR)
unzip("/mnt/c/Users/zd200/OneDrive/Learning/ML/svm_light.zip")
download.file('http://download.joachims.org/svm_light/current/svm_light_linux64.tar.gz',"svm_light_linux64.tar.gz")
untar('svm_light_linux64.tar.gz')
getwd()
model.light <- svmlight(churn~., data = trainset,
                        kernel="radial", cost=1,gamma=1/ncol(trainset),pathsvm="/mnt/c/Users/zd200/Desktop/Metagenomics/PJS")

'
Parameters:
  SVM-Type:  C-classification 
SVM-Kernel:  radial 
cost:  1 

Number of Support Vectors:  959

( 546 413 )


Number of Classes:  2 

Levels: 
  yes no'
# 选择惩罚因子
# 给iris数据集取子集，用于建模
iris.subset <- subset(iris, select=c("Sepal.Length", "Sepal.Width","Species"),
                      Species %in% c("setosa","virginica"))
# 绘制散点图
plot(x=iris.subset$Sepal.Length, y=iris.subset$Sepal.Width,
     col=iris.subset$Species,pch=1)
# 将惩罚因子设置为1
svm.model <- svm(Species~., data = iris.subset,
                        kernel="linear", cost=1,scale=FALSE)
# 标注支持向量
points(iris.subset[svm.model$index, c(1,2)], col="blue", cex=2)
# 加分隔线
w <- t(svm.model$coefs) %*% svm.model$SV
b <- -svm.model$rho
abline(a=-b/w[1,2], b=-w[1,1]/w[1,2],col="red", lty=5)
# 惩罚因子1000
# 绘制散点图
plot(x=iris.subset$Sepal.Length, y=iris.subset$Sepal.Width,
     col=iris.subset$Species,pch=1)
svm.model <- svm(Species~., data = iris.subset,
                 kernel="linear", cost=1000,scale=FALSE)
# 标注支持向量
points(iris.subset[svm.model$index, c(1,2)], col="blue", cex=2)
# 加分隔线
w <- t(svm.model$coefs) %*% svm.model$SV
b <- -svm.model$rho
abline(a=-b/w[1,2], b=-w[1,1]/w[1,2],col="red", lty=5)

# SVM可视化
data(iris)
model.iris <- svm(Species ~., iris)
plot(model.iris, iris, Petal.Width~Petal.Length, slice = 
       list(Sepal.Width = 3, Sepal.Length = 4))
plot(model, trainset, total_day_minutes ~ total_intl_charge)
# 预测分类
svm.pred <- predict(model, testset[,!names(testset) %in% c("churn")])
svm.table <- table(svm.pred, testset$churn)
svm.table
# 调用classAgreement计算分类一致性
classAgreement(svm.table)
# 混淆矩阵
library(caret)
confusionMatrix(svm.table)
```
![](https://upload-images.jianshu.io/upload_images/6644753-324cc4419d90b551.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-22878c3b29ca9578.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-d972ebd66adb7273.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-a0a962ac898f650d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
两幅图的比较说明，惩罚因子有较大影响。
![](https://upload-images.jianshu.io/upload_images/6644753-79926151e9c5feff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
svm.pred  yes   no
     yes  106   14
     no   108 1293
$diag
[1] 0.9197896

$kappa
[1] 0.5936486

$rand
[1] 0.8523496

$crand
[1] 0.5342531

Confusion Matrix and Statistics

        
svm.pred  yes   no
     yes  106   14
     no   108 1293
                                         
               Accuracy : 0.9198         
                 95% CI : (0.905, 0.9329)
    No Information Rate : 0.8593         
    P-Value [Acc > NIR] : 2.218e-13      
                                         
                  Kappa : 0.5936         
                                         
 Mcnemar's Test P-Value : < 2.2e-16      
                                         
            Sensitivity : 0.49533        
            Specificity : 0.98929        
         Pos Pred Value : 0.88333        
         Neg Pred Value : 0.92291        
             Prevalence : 0.14070        
         Detection Rate : 0.06969        
   Detection Prevalence : 0.07890        
      Balanced Accuracy : 0.74231        
                                         
       'Positive' Class : yes 
```
**扩展**
```
# regression
library(car)
library(e1071)
data("Quartet")
model.regression <- svm(Quartet$y1~ Quartet$x, type="eps-regression")
predict.y <- predict(model.regression, Quartet$x)
predict.y
# ############
       1        2        3        4        5        6        7        8        9       10       11 
8.196894 7.152946 8.807471 7.713099 8.533578 8.774046 6.186349 5.763689 8.726925 6.621373 5.882946 
plot(Quartet$x, Quartet$y1,pch=19)
points(Quartet$x, predict.y, pch=15, col='red')
```
![](https://upload-images.jianshu.io/upload_images/6644753-7b3b8102ccf5a9c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 6.6 调整支持向量
除了选择不同的特征集和核函数，还可以借助参数gamma以及惩罚因子来调整支持向量机的性能。tune.svm函数简化了这个过程。
```
# tuned
tuned <- tune.svm(churn~., data = trainset, gamma = 10^(-6:-1),
                   cost = 10^(1:2))
summary(tuned)
# ###########
Parameter tuning of ‘svm’:

- sampling method: 10-fold cross validation 

- best parameters:
 gamma cost
  0.01  100

- best performance: 0.07272516 
model.tuned <- svm(churn~., data = trainset, gamma =tuned$best.parameters$gamma,
                   cost = tuned$best.parameters$cost)
summary(model.tuned)
all:
svm(formula = churn ~ ., data = trainset, gamma = tuned$best.parameters$gamma, cost = tuned$best.parameters$cost)


Parameters:
   SVM-Type:  C-classification 
 SVM-Kernel:  radial 
       cost:  100 

Number of Support Vectors:  768

 ( 424 344 )


Number of Classes:  2 

Levels: 
 yes no
# predict
svm.tuned.pred <- predict(model.tuned, testset[,!names(testset) %in% c("churn")])
svm.tuned.table <- table(svm.tuned.pred, testset$churn)
svm.tuned.table
#性能评测
classAgreement(svm.tuned.table)
$diag
[1] 0.9250493

$kappa
[1] 0.6504167

$rand
[1] 0.8612426

$crand
[1] 0.5882917

> confusionMatrix(svm.tuned.table)
Confusion Matrix and Statistics

              
svm.tuned.pred  yes   no
           yes  128   28
           no    86 1279
                                          
               Accuracy : 0.925           
                 95% CI : (0.9106, 0.9378)
    No Information Rate : 0.8593          
    P-Value [Acc > NIR] : 1.027e-15       
                                          
                  Kappa : 0.6504          
```
tuned.svm采用十折交叉来获得每次组合的错误偏差，选择误差最低的最佳参数组合。使用这个组合再训练一个支持向量机。
# 6.7 neuralnet包训练神经网络
我们一般认为神经网络是非常高技术含量的东西，这里我们就学习下这个“高大上”的东西。其实，应该深度学习的技术含量高点，神经网络应该推出好多好多年了。
```
############################神经网络
data(iris)
ind <- sample(2,nrow(iris),replace = TRUE,
              prob = c(0.7,0.3))
trainset <- iris[ind==1,]
testset <- iris[ind==2,]
install.packages("neuralnet")
library(neuralnet)
trainset$setosa <- trainset$Species =='setosa'
trainset$virginica <- trainset$Species=="virginica"
trainset$versicolor <- trainset$Species=='versicolor'
network <- neuralnet(versicolor + virginica + setosa ~ Sepal.Length + Sepal.Width + Petal.Length + Petal.Width, trainset,
                     hidden = 3)

```
神经元的优势是可检测非线性关系，利用算法的并行化实现对大数据集的高效训练，无参模型，避免参数估计中的错误。不足是容易陷入局部最优，算法训练时间过长，可能过拟合。
# 6.8 可视化
```
# plot 
plot(network)
# 可视化泛化权值
# 可视化泛化权值
par(mfrow=c(2,2))
gwplot(network,selected.covariate = "Petal.Width")
gwplot(network,selected.covariate = "Sepal.Width")
gwplot(network,selected.covariate = "Petal.Length")
gwplot(network,selected.covariate = "Sepal.Length")
```
![图看起来不错](https://upload-images.jianshu.io/upload_images/6644753-da55bb3a4b505182.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-7c3367e832b80245.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
图中泛化权值接近于0，说明协变量对分类结果影响不大，若总体方差>1，则协变量对分类结果存在非线性影响。     
# 6.9 基于neuralnet包得到的模型实现类标号预测
```
# 类标号预测
net.predict <- compute(network, testset[-5])$net.result
net.predicttion <- c('setosa', "virginica", 'versicolor')[apply(net.predict,1,which.max)]
predict.table <- table(testset$Species, net.predicttion)
predict.table
            net.predicttion
             setosa versicolor virginica
  setosa          0         14         0
  versicolor      8          0         1
  virginica       2          0        12
library(e1071)
classAgreement(predict.table)
$diag
[1] 0.3243243

$kappa
[1] -0.004343105

$rand
[1] 0.9099099

$crand
[1] 0.7944529
confusionMatrix(predict.table)
Confusion Matrix and Statistics

            net.predicttion
             setosa versicolor virginica
  setosa          0         14         0
  versicolor      8          0         1
  virginica       2          0        12

Overall Statistics
                                          
               Accuracy : 0.3243          
                 95% CI : (0.1801, 0.4979)
    No Information Rate : 0.3784          
    P-Value [Acc > NIR] : 0.8004          
                                          
                  Kappa : -0.0043 
```
compute函数还可以获得每一层的输出`compute(network, testset[-5])`。
# 6.10 nnet包训练神经模型
这个包提供了传统的前馈反向传播神经网络算法的功能实现，neuralnet包实现了大部分神经网络算法。
```
# ####nnet
install.packages('nnet')
library(nnet)
# 利用前面分好的训练和测试集 隐藏单元size，初始随机数rang，权值衰减参数decay， 最大迭代次数maxit
iris.nn <- nnet(Species~.,data = trainset[,-c(6,7,8)], size = 2, rang = 0.1, decay = 5e-4, maxit = 200)
summary(iris.nn)
a 4-2-3 network with 19 weights
options were - softmax modelling  decay=5e-04
```
# 6.11 基于nnet模型实现类预测
```
 library(caret)
iris.predict <- predict(iris.nn, testset[,-c(6,7,8)], type = "class")
m.table <- table(testset$Species, iris.predict)
confusionMatrix(m.table)
confusionMatrix(m.table)
Confusion Matrix and Statistics

            iris.predict
             setosa versicolor virginica
  setosa         14          0         0
  versicolor      0          8         1
  virginica       0          2        12

Overall Statistics
                                         
               Accuracy : 0.9189         
                 95% CI : (0.7809, 0.983)
    No Information Rate : 0.3784         
    P-Value [Acc > NIR] : 8.776e-12      
                                         
                  Kappa : 0.8768  
```
如果不指定type=class，默认输出概率矩阵。
```
head(predict(iris.nn, testset[,-c(6,7,8)]))
      setosa   versicolor    virginica
3  0.9995652 0.0004348076 4.156404e-13
4  0.9995187 0.0004813482 4.637527e-13
5  0.9995810 0.0004190092 3.993965e-13
7  0.9995603 0.0004397184 4.206989e-13
17 0.9995906 0.0004094486 3.895892e-13
19 0.9995774 0.0004225847 4.030688e-13
```
