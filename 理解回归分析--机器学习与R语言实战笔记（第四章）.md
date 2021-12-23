回归是一种有监督的学习方式，用于建模分析一个独立变量（响应变量）和一个或多个非独立变量（预测变量）之间的关联。
```
library(car)
data("Quartet")
str(Quartet)
# #############################
'data.frame':	11 obs. of  6 variables:
 $ x : int  10 8 13 9 11 14 6 4 12 7 ...
 $ y1: num  8.04 6.95 7.58 8.81 8.33 ...
 $ y2: num  9.14 8.14 8.74 8.77 9.26 8.1 6.13 3.1 9.13 7.26 ...
 $ y3: num  7.46 6.77 12.74 7.11 7.81 ...
 $ x4: int  8 8 8 8 8 8 8 19 8 8 ...
 $ y4: num  6.58 5.76 7.71 8.84 8.47 7.04 5.25 12.5 5.56 7.91 ...
plot(Quartet$x,Quartet$y1)
lmfit<- lm(y1~x, Quartet)
abline(lmfit, col="red")
lmfit
# #################
Call:
lm(formula = y1 ~ x, data = Quartet)

Coefficients:
(Intercept)            x  
     3.0001       0.5001  
# 预测，95%置信区间
predict(lmfit,data.frame(x=c(3,4,5)), interval = 'confidence', level=0.95)
       fit      lwr      upr
1 4.500364 2.691375 6.309352
2 5.000455 3.422513 6.578396
3 5.500545 4.140531 6.860560
# 预测，预测区间
predict(lmfit,data.frame(x=c(3,4,5)), interval = 'predict')
       fit      lwr      upr
1 4.500364 1.169022 7.831705
2 5.000455 1.788711 8.212198
3 5.500545 2.390073 8.611017
```
![](https://upload-images.jianshu.io/upload_images/6644753-ed43fe60b6bfe7d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
lsfit函数同样可以进行简单的线性回归分析。
summay函数可以给出摘要统计信息, 仅仅依靠R^2不能得出回归模型是否符合要求，往往使用经过调整的R^2进行无偏差的估计。
**扩展**
也可以通过coffcients()参数, cofint(lmfit,level=0.95)置信区间, fitted()#结果, residuals(), anova(), vcov(), 回归拟合诊断influence()来获取模型的性质。
# 生成模型的诊断图
```
par(mfrow=c(2,2))
plot(lmfit)
```
![](https://upload-images.jianshu.io/upload_images/6644753-30beab4ad6193052.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
左上，残差和拟合值的关联；右上，残差正态图；左下，位置-尺度图，残差和拟合值的平方根；右下，残差与杠杆值，杠杆值是衡量观测点对回归效果影响大小的度量，是观测点到回归中心的距离以及孤立级别。Cook距离，测量某个观测值对一组回归系数的影响。
`plot(cooks.distance(lmfit))`
![](https://upload-images.jianshu.io/upload_images/6644753-6026f7cb24f3d630.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# lm多项式回归模型
```
plot(Quartet$x, Quartet$y2)
lmfit <- lm(Quartet$y2~poly(Quartet$x,2))
fi <-  lm(Quartet$y2~I(Quartet$x)+I(Quartet$x^2)) #同样的效果
lines(sort(Quartet$x),lmfit$fit[order(Quartet$x)],col='red')
```
![](https://upload-images.jianshu.io/upload_images/6644753-c8caa6725c820f5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-2fb2e921a6f00708.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
抛物线
# rlm函数生成稳健线性回归模型
```
plot(Quartet$x, Quartet$y3)
library(MASS)
lmfit<- rlm(Quartet$y3~Quartet$x)
abline(lmfit,col='red')
```
![](https://upload-images.jianshu.io/upload_images/6644753-5a98cc88579db4a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/6644753-0c35d5804d519d85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
看起来是舍弃了极值
# 在SLID数据集上研究线性回归案例
```
library(car)
str(SLID)
# 'data.frame':	7425 obs. of  5 variables:
#   $ wages    : num  10.6 11 NA 17.8 NA ...
# $ education: num  15 13.2 16 14 8 16 12 14.5 15 10 ...
# $ age      : int  40 19 49 46 71 50 70 42 31 56 ...
# $ sex      : Factor w/ 2 levels "Female","Male": 2 2 2 2 2 1 1 1 2 1 ...
# $ language : Factor w/ 3 levels "English","French",..: 1 1 

par(mfrow=c(2,2))
plot(SLID$wages~SLID$language)
plot(SLID$wages~SLID$age)
plot(SLID$wages~SLID$education)
plot(SLID$wages~SLID$sex)

lmfit<- lm(wages ~.,data = SLID)
summary(lmfit)
# Call:
#   lm(formula = wages ~ ., data = SLID)
# 
# Residuals:
#   Min      1Q  Median      3Q     Max 
# -26.062  -4.347  -0.797   3.237  35.908 
# 
# Coefficients:
#   Estimate Std. Error t value Pr(>|t|)    
# (Intercept)    -7.888779   0.612263 -12.885   <2e-16 ***
#   education       0.916614   0.034762  26.368   <2e-16 ***
#   age             0.255137   0.008714  29.278   <2e-16 ***
#   sexMale         3.455411   0.209195  16.518   <2e-16 ***
#   languageFrench -0.015223   0.426732  -0.036    0.972    
# languageOther   0.142605   0.325058   0.439    0.661    
# ---
#   Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
# 
# Residual standard error: 6.6 on 3981 degrees of freedom
# (3438 observations deleted due to missingness)
# Multiple R-squared:  0.2973,	Adjusted R-squared:  0.2964 
# F-statistic: 336.8 on 5 and 3981 DF,  p-value: < 2.2e-16
# 发现language不显著，去除,F统计量提升至565
lmfit<- lm(wages ~age+sex+education,data = SLID)
summary(lmfit)
par(mfrow=c(2,2))
plot(lmfit)
# wages变化范围较大，为对称，log之
lmfit<- lm(log(wages) ~age+sex+education,data = SLID)
plot(lmfit)
vif(lmfit) # 共线性回归情况
      age       sex education 
 1.011613  1.000834  1.012179 
sqrt(vif(lmfit)) >2
      age   sexMale education 
    FALSE     FALSE     FALSE 
install.packages("lmtest")
library(lmtest)
bptest(lmfit) # 诊断异方差性

	studentized Breusch-Pagan test

data:  lmfit
BP = 29.031, df = 3, p-value = 2.206e-06
install.packages("rms")
library(rms)
olsfit<-ols(log(wages) ~age+sex+education,data = SLID,x=TRUE,y=TRUE)
robcov(olsfit) # robcov函数修正标准误

```
画了四幅比较朴素的图。
![](https://upload-images.jianshu.io/upload_images/6644753-d08ad2ec0fa20684.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-1517bea84b8f2561.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-56f5501faa8b1025.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
一般线性回归，假设观测值的方差或误差是常数或者齐次，异方差是指方差分布不均匀，导致评估标准差存在偏差。
# 基于高斯模型的广义线性回归
广义线性模型是对线性回归的推广，模型通过一个连接函数得到线性预测结果。本书是一本难得的写的内容很深入的书，阅读到此已经深有体会。默认情况下glm的族对象是高斯模型，和lm功能一致。
```
lmfit1 <- glm(wages ~age+sex+education,data = SLID,family = gaussian)
summary(lmfit1)
lmfit2<- lm(wages ~age+sex+education,data = SLID)
anova(lmfit1,lmfit2)
# ################
Model: gaussian, link: identity

Response: wages

Terms added sequentially (first to last)


          Df Deviance Resid. Df Resid. Dev
NULL                       4013     248686
age        1    31953      4012     216733
sex        1    11074      4011     205659
education  1    30883      4010     174776
```
以上可以看出，两个模型基本是相同的。
# 基于泊松模型的广义线性回归
假设变量服从泊松分布时，可以采用对数线性模型来拟合计数数据。这个数据集是织布机的异常数据。
```
data("warpbreaks")
head(warpbreaks)
# breaks wool tension
# 1     26    A       L
# 2     30    A       L
# 3     54    A       L
# 4     25    A       L
# 5     70    A       L
# 6     52    A       L
summary(glm(breaks~tension, family = "poisson",data = warpbreaks))
# #################
Call:
glm(formula = breaks ~ tension, family = "poisson", data = warpbreaks)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-4.2464  -1.6031  -0.5872   1.2813   4.9366  

Coefficients:
            Estimate Std. Error z value Pr(>|z|)    
(Intercept)  3.59426    0.03907  91.988  < 2e-16 ***
tensionM    -0.32132    0.06027  -5.332 9.73e-08 ***
tensionH    -0.51849    0.06396  -8.107 5.21e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for poisson family taken to be 1)

    Null deviance: 297.37  on 53  degrees of freedom
Residual deviance: 226.43  on 51  degrees of freedom
AIC: 507.09

Number of Fisher Scoring iterations: 4
```
# 基于二项模型的广义线性模型
二项分布，响应变量的每个观测值为0或1。
```
summary(glm(vs ~ hp + mpg + gear, data = mtcars, family = binomial))
# ########################
Call:
glm(formula = vs ~ hp + mpg + gear, family = binomial, data = mtcars)

Deviance Residuals: 
     Min        1Q    Median        3Q       Max  
-1.68166  -0.23743  -0.00945   0.30884   1.55688  

Coefficients:
            Estimate Std. Error z value Pr(>|z|)  
(Intercept) 11.95183    8.00322   1.493   0.1353  
hp          -0.07322    0.03440  -2.129   0.0333 *
mpg          0.16051    0.27538   0.583   0.5600  
gear        -1.66526    1.76407  -0.944   0.3452  
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 43.860  on 31  degrees of freedom
Residual deviance: 15.651  on 28  degrees of freedom
AIC: 23.651

Number of Fisher Scoring iterations: 7

```
如果希望选择别的连接函数，可以在调用时增加link参数的设置。
`summary(glm(vs ~ hp + mpg + gear, data = mtcars, family = binomial(link='probit')))`
# 利用广义加性模型处理数据
广义加性模型GAM是一般线性模型的半参数扩展，更适合处理那些非独立变量与独立变量之间存在复杂非线性关系的情况。设计用于最大化来自不同分布的非独立变量y的预测能力，评估预测变量的非参数函数。
```
install.packages("mgcv") # gam函数在这个包
library(mgcv)
install.packages("MASS")
library(MASS)
attach(Boston) # 城郊房价数据集
str(Boston)
fit<-gam(dis ~s(nox)) # nox 独立变量
summary(fit)
# ####################
Family: gaussian 
Link function: identity 

Formula:
dis ~ s(nox)

Parametric coefficients:
            Estimate Std. Error t value Pr(>|t|)    
(Intercept)  3.79504    0.04507   84.21   <2e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Approximate significance of smooth terms:
         edf Ref.df   F p-value    
s(nox) 8.434  8.893 189  <2e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

R-sq.(adj) =  0.768   Deviance explained = 77.2%
GCV = 1.0472  Scale est. = 1.0277    n = 506
```
mgcv包的bam函数，适合构建大数据集的广义加性模型，需要的内存更少，效率更高。
# 可视化广义加性模型
又到了大家最爱的画图环节,简单几行就出图啦。
```
plot(nox,dis)
x=seq(0,1,length=500)
y=predict(fit,data.frame(nox=x)) # 预测值
lines(x,y, col='red',lwd=2) # 回归线
plot(fit)
```
![](https://upload-images.jianshu.io/upload_images/6644753-44d634ea99627a8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-3ff8172759f05e0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/6644753-1d3e54bca0cf2400.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 诊断广义加性模型
gam.check绘制诊断图
两行解决，又是4个图
```
par(mfrow=c(2,2))
gam.check(fit)
# ###############
Method: GCV   Optimizer: magic
Smoothing parameter selection converged after 7 iterations. #平滑参数迭代7次收敛
The RMS GCV score gradient at convergence was 8.79622e-06 .
The Hessian was positive definite.
Model rank =  10 / 10 

Basis dimension (k) checking results. Low p-value (k-index<1) may
indicate that k is too low, especially if edf is close to k'. # 参数k初始值偏低

         k'  edf k-index p-value    
s(nox) 9.00 8.43     0.4  <2e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
```

左上，分位数比较图，判断孤立点和重尾；左上，残差和线性预测值，发现非常数的误差方差；左下是残差的直方图，发现非正态分布；右下为响应和拟合值图。
