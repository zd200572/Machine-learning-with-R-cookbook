# 挖掘RMS Titanic数据集
## 六个阶段
提出问题、数据采集、数据清洗、基础数据分析、高级分析和模型评估
直接上代码呀！
 数据下载，需要科学地上网[下载地址](https://storage.googleapis.com/kagglesdsdata/competitions/3136/26502/train.csv?GoogleAccessId=web-data@kaggle-161607.iam.gserviceaccount.com&Expires=1626930204&Signature=X%2FiDp2Z5s0KwLmM%2BqBmlmmanMz6CBJmpMPo9Yv3jvWrNF9KKz0ThQo906VR7AP1sSDmcsWNVHyUE%2FLkclmOlu922FFDWfZn6%2BYXSADMJ2wqbT2JAQV5vLG%2Bx4JDYse%2F6Ah5osN5ME3oMEjf%2BDTZ%2Fl13jmn5spkqgB0NIx%2FGrUs36YLx49aXCRozJ15iDSyI7j1yA%2FRRtZyhFKpAZUg%2B6nAkrR3t7k1rVZJpJwhZ3YlTIha8vg%2BbXbhrjeYdKbzE3gYn4rQ%2Bu%2FTWhWMyWyqac6Db8dItSzeyiK0lZmq8ApKL%2FBGtqR2nnZ%2BPzMXjw99evfnp60DnbV9Lew1RT7iAXcQ%3D%3D&response-content-disposition=attachment%3B+filename%3Dtrain.csv)
```
# 数据下载
train.data <- read.csv("train.csv", na.strings = c("NA", ""))
# 类型转换，分类变量转因子
train.data$Survived <- factor(train.data$Survived)
train.data$Pclass <- factor(train.data$Pclass)
# 检测缺失
sum(is.na(train.data$Age) == TRUE)/length(train.data$Age)
sapply(train.data, function(df){
  sum(is.na(df)==TRUE)/length(df)
})
# 缺失数据可视化
# install.packages("Amelia")
require(Amelia)
missmap(train.data, main = "缺失数据图")
```
![](https://upload-images.jianshu.io/upload_images/6644753-feb078495ad67ba7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
AmeliaView()
```
![](https://upload-images.jianshu.io/upload_images/6644753-cd07c51f3aa2654c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
AmeliaView有交互式的GUI，赞一个！
![](https://upload-images.jianshu.io/upload_images/6644753-5e674486323dc1a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
table(train.data$Embarked,useNA = "always")
# C    Q    S <NA> 
 168   77  644    2
# 将缺失处理为理可能结果
train.data$Embarked[which(is.na(train.data$Embarked))] <- 'S'
table(train.data$Embarked,useNA = "always")
#  C    Q    S <NA> 
 168   77  646    0
#　获得不同称呼类别
train.data$Name <- as.character(train.data$Name)
# 先用空白标记
table_words <- table(unlist(strsplit(train.data$Name, "\\s+")))
str(table_words)
# 'table' int [1:1673(1d)] 1 1 1 1 1 1 1 1 1 1 ...
# - attr(*, "dimnames")=List of 1
# ..$ : chr [1:1673] "\"Andy\"" "\"Annie" "\"Annie\"" "\"Archie\"" ...
sort(table_words [grep('\\.',names(table_words))],
     decreasing = TRUE)
# Mr.     Miss.      Mrs.   Master.       Dr.      Rev. 
      517       182       125        40         7         6 
     Col.    Major.     Mlle.     Capt. Countess.      Don. 
        2         2         2         1         1         1 
Jonkheer.        L.     Lady.      Mme.       Ms.      Sir. 
        1         1         1         1         1         1 
```
# 识别和可视化

![](https://upload-images.jianshu.io/upload_images/6644753-9be8a43014ee413c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-40fc887fcac5b4f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-9048926af9ea30b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-5df7e019ff3aac60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-6918c8a0cf6529f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-57924976d96c7fee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-191417770f3b32ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-f7775804ed8ad34f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-4ee012eb67a6aa30.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-e82532418da60cb6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-0af089bc7f85d66f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-4f0cc0e80d8903ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-e928ed07836edc40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 ![](https://upload-images.jianshu.io/upload_images/6644753-d4350500924fd011.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-d32a592c786e4594.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 决策树构建
```
	 Conditional inference tree with 6 terminal nodes

Response:  Survived 
Inputs:  Pclass, Sex, Age, SibSp, Fare, Parch, Embarked 
Number of observations:  623 

1) Sex == {female}; criterion = 1, statistic = 180.207
  2) Pclass == {3}; criterion = 1, statistic = 59.081
    3)*  weights = 92 
  2) Pclass == {1, 2}
    4)*  weights = 111 
1) Sex == {male}
  5) Pclass == {1}; criterion = 1, statistic = 21.732
    6) Age <= 38; criterion = 0.99, statistic = 10.105
      7)*  weights = 57 
    6) Age > 38
      8)*  weights = 39 
  5) Pclass == {2, 3}
    9) Age <= 3; criterion = 1, statistic = 22.354
      10)*  weights = 12 
    9) Age > 3
      11)*  weights = 312
```
![](https://upload-images.jianshu.io/upload_images/6644753-27174df95c5ef453.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

party包的决策树，与rpart包的相比，可以避免rpart包在变量选择时的倾斜，并且更倾向于选择能够产生更多分支或缺失值多的变量。

```
Confusion Matrix and Statistics

          Reference
Prediction   0   1
         0 146  57
         1   7  57
                                          
               Accuracy : 0.7603          
                 95% CI : (0.7045, 0.8102)
    No Information Rate : 0.573           
    P-Value [Acc > NIR] : 1.279e-10       
                                          
                  Kappa : 0.4811          
                                          
 Mcnemar's Test P-Value : 9.068e-10       
                                          
            Sensitivity : 0.9542          
            Specificity : 0.5000          
         Pos Pred Value : 0.7192          
         Neg Pred Value : 0.8906          
             Prevalence : 0.5730          
         Detection Rate : 0.5468          
   Detection Prevalence : 0.7603          
      Balanced Accuracy : 0.7271          
                                          
       'Positive' Class : 0 
```
![](https://upload-images.jianshu.io/upload_images/6644753-babcdad34c7fca6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
