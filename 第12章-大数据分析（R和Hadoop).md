
RHadoop是R支持Hadoop大数据分析和处理提供的算法包合集。传统统计学主要关注样本数据（小数据集）的分析，可能忽略发生概率极小单导致不确定性的结果。当数据量大到一台机器无法处理时，只能求助于超算或者Hadoop这样的可扩展方案。Hadoop是最流行的一种开源可扩展大数据处理基础架构，基于集群并行数据存储和计算。RHadoop主要包含五个算法包：
- rmr：R-MapReduce交互接口，我们只需关注map和reduce函数。
- rhdfs：R-HDFS交互接口，访问HDFS的数据。
- rhbase：R-HBase的交互接口，操纵存储在HBase中的表格。
- plyrmr：MapReduce的高级抽象，支持勒plyr语法实现常规数据操作。
- ravro： 读写avro文件，与HDFS数据交换。
# 准备RHadoop环境
使用这个虚拟机啦，这个公司好像已经停止提供相应镜像了，找到一个书中提到的mapr的。
https://package.mapr.com/releases/v6.1.0/sandbox/MapR-Sandbox-For-Hadoop-6.1.0.ova
下载Vmware，然后导入这个虚拟机，试用就够了。用户名和密码都是mapr
[Installing the Sandbox on VirtualBox (hpe.com)](https://docs.datafabric.hpe.com/61/SandboxHadoop/t_install_sandbox_vbox.html)
这里安装吃了很多苦呀，折腾了几个晚上，终于发现conda安装是最省事的，这个包系列已经6年没有更新了，对新python的兼容性很差，难道hadoop已经衰落，还是工业环境只要稳定能用就好，不需要再更新呢？
```
# 虚拟机已经有端口转发，直接连接 ssh mapr@localhost -p 2222
su
mapr
yum install R-core -y
R
# 安装这里走了许多弯路，发现最简单的方式是conda，简直万能的，节约很多时间
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
source /root/miniconda3/bin/activate
 conda install r-rmr2 r-devtools
conda install -c mndrake r-rhdfs -y
vi /root/mapr/word.txt
i
123
:wq
# 解决两个java环境相互冲突的问题
Error: package or namespace load failed for ‘rJava’:
 .onLoad failed in loadNamespace() for 'rJava', details:
  call: dyn.load(file, DLLpath = DLLpath, ...)
  error: unable to load shared object '/usr/lib64/R/library/rJava/libs/rJava.so':
  libjvm.so: cannot open shared object file: No such file or directory
vi /root/.bashrc
i
export JAVA_HOME=/root/miniconda3
:wq
source /root/.bashrc
```
# 12.5 在rhdfs中操作HDFS
```
source /root/miniconda3/bin/activate
R
# 以下两个变量可以放入.rprofile文件，这样就不需要每次运行了。
Sys.setenv(HADOOP_CMD='/usr/bin/hadoop')
Sys.setenv(HADOOP_STREAMING='/opt/mapr/hadoop/hadoop-2.7.0/share/hadoop/tools/lib/hadoop-streaming-2.7.0-mapr-1808.jar')
library(rhdfs)
hdfs.init()
# 复制文件到HDFS
hdfs.put('word.txt', './')
[1] TRUE
# 查看
hdfs.ls('./')
  permission owner group size          modtime                file
1 drwxr-xr-x  mapr  mapr    1 2018-10-23 17:55      /user/mapr/tmp
2 -rwxr-xr-x  mapr  mapr    4 2021-12-15 10:01 /user/mapr/word.txt
hdfs.copy('word.txt', 'wordcnt.txt')
[1] TRUE
hdfs.move('wordcnt.txt','./data/wordcnt.txt')
[1] TRUE
hdfs.delete('./data')
Deleted maprfs:///user/mapr/data
[1] TRUE
hdfs.rm('wordcnt.txt')
Deleted maprfs:///user/mapr/wordcnt.txt
[1] TRUE
# 下载
hdfs.get('word.txt', 'test.txt')
[1] TRUE
hdfs.rename('word.txt','2.txt')
[1] TRUE
hdfs.chmod('2.txt', permissions='777')
hdfs.file.info('./')
      perms isDir     block replication owner group size              modtime
1 rwxr-xr-x  TRUE 268435456           1  mapr  mapr    2 53926-08-07 05:11:00
  path
1   ./
# 写HDFS
f <- hdfs.file('iris.txt', 'w')
data(iris)
hdfs.write(iris,f)
hdfs.close(f)
[1] TRUE
# 读HDFS
f <- hdfs.file('iris.txt', 'r')
dfserialized <- hdfs.read(f)
df <- unserialize(dfserialized)
head(df)
head(df)
  Sepal.Length Sepal.Width Petal.Length Petal.Width Species
1          5.1         3.5          1.4         0.2  setosa
2          4.9         3.0          1.4         0.2  setosa
3          4.7         3.2          1.3         0.2  setosa
4          4.6         3.1          1.5         0.2  setosa
5          5.0         3.6          1.4         0.2  setosa
6          5.4         3.9          1.7         0.4  setosa
```
寻找streaming可以用以下命令：
```
locate streaming|grep jar |more
/opt/mapr/hadoop/hadoop-2.7.0/share/hadoop/tools/lib/hadoop-streaming-2.7.0-mapr-1808.jar
/opt/mapr/hadoop/hadoop-2.7.0/share/hadoop/tools/sources/hadoop-streaming-2.7.0-mapr-1808-sources.jar
/opt/mapr/hadoop/hadoop-2.7.0/share/hadoop/tools/sources/hadoop-streaming-2.7.0-mapr-1808-test-sources.jar
/opt/mapr/hive/hive-2.3/hcatalog/share/hcatalog/hive-hcatalog-streaming-2.3.3-mapr-1808.jar
/opt/mapr/oozie/oozie-4.3.0/lib/oozie-sharelib-streaming-4.3.0-mapr-1808.jar
/opt/mapr/oozie/oozie-4.3.0/oozie-server/webapps/oozie/WEB-INF/lib/oozie-sharelib-streaming-4.3.0-mapr-1808.jar
/opt/mapr/oozie/oozie-4.3.0/share/lib/mapreduce-streaming/commons-io-2.4.jar
/opt/mapr/oozie/oozie-4.3.0/share/lib/mapreduce-streaming/hadoop-streaming-2.7.0-mapr-1808.jar
/opt/mapr/oozie/oozie-4.3.0/share/lib/mapreduce-streaming/oozie-sharelib-streaming-4.3.0-mapr-1808.jar
/opt/mapr/oozie/oozie-4.3.0/share/lib/spark/spark-streaming-kafka-0-9_2.11-2.3.1-mapr-1808.jar
/opt/mapr/oozie/oozie-4.3.0/share/lib/spark/spark-streaming_2.11-2.3.1-mapr-1808.jar
/opt/mapr/pig/pig-0.16/test/e2e/pig/lib/hadoop-0.23.0-streaming.jar
/opt/mapr/pig/pig-0.16/test/e2e/pig/lib/hadoop-streaming.jar
/opt/mapr/spark/spark-2.3.1/jars/spark-streaming-kafka-0-9_2.11-2.3.1-mapr-1808.jar
/opt/mapr/spark/spark-2.3.1/jars/spark-streaming_2.11-2.3.1-mapr-1808.jar
# 另外，如果想操作更方便，可以用rstudio-server，虚拟机要配置相应端口转发
wget https://download2.rstudio.org/server/centos8/x86_64/rstudio-server-rhel-2021.09.1-372-x86_64.rpm
sudo yum install rstudio-server-rhel-2021.09.1-372-x86_64.rpm
```
# 12.6 RHadoop中解决单词计数问题
```
# 准备数据
https://gitee.com/zd200572/ml_R_cookbook.git
library(rmr2)
hdfs.init()
21/12/15 10:27:05 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
# 创建目录并放入文件
hdfs.mkdir("/user/mapr/wordcount/data")
[1] TRUE
hdfs.put("ml_R_cookbook/CH12/wc_input.txt", '/user/mapr/wordcount/data')
[1] TRUE
# map函数
map <- function(.,lines){ keyval(
  unlist(
    strsplit(
      x <- lines,
      split <- " +"    )
  ),
1)}
# reduce函数
reduce <- function(word, counts) {
  keyval(word, sum(counts))
}
hdfs.root <- 'wordcount'
hdfs.data <- file.path(hdfs.root, 'data')
hdfs.out <- file.path(hdfs.root, 'out2')
wordcounts <- function(input, output=NULL){
  mapreduce(input=input, output=output, input.format="text",
  map=map, reduce=reduce)
}
out <- wordcounts(hdfs.data, hdfs.out)
hdfs.remove('')
results <- from.dfs(out)
results$key[order(results$eval), decreasing=TRUE][1:10]
# 调用map完成单词计数
# 还是报错，这个错就hold不住啦，看资源还凑活充足呢,或许硬件要求高！
21/12/18 01:13:05 INFO mapreduce.Job: Counters: 19
        Job Counters 
                Failed map tasks=7
                Killed map tasks=1
                Killed reduce tasks=1
                Launched map tasks=8
                Other local map tasks=6
                Data-local map tasks=2
                Total time spent by all maps in occupied slots (ms)=209232
                Total time spent by all reduces in occupied slots (ms)=0
                Total time spent by all map tasks (ms)=52308
                Total time spent by all reduce tasks (ms)=0
                Total vcore-seconds taken by all map tasks=52308
                Total vcore-seconds taken by all reduce tasks=0
                Total megabyte-seconds taken by all map tasks=53563392
                Total megabyte-seconds taken by all reduce tasks=0
                DISK_MILLIS_MAPS=26157
                DISK_MILLIS_REDUCES=0
        Map-Reduce Framework
                CPU time spent (ms)=0
                Physical memory (bytes) snapshot=0
                Virtual memory (bytes) snapshot=0
21/12/18 01:13:05 ERROR streaming.StreamJob: Job not successful!
Streaming Command Failed!
Error in mr(map = map, reduce = reduce, combine = combine, vectorized.reduce,  : 
  hadoop streaming failed with error code 1
```
![hadoop监控](https://upload-images.jianshu.io/upload_images/6644753-79bce7b3d8cadae6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![任务监控](https://upload-images.jianshu.io/upload_images/6644753-393df2e88d7ae131.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
深深地感受到了大数据的门槛还是挺高的，特别是好些软件不够完善，对新手不够友好的情况下，还需要大量的计算资源做支撑。如果没法实践，那就理解下具体过程吧：
MapReduce程序被分成map和reduce两部分，map函数先使用strsplit函数将一行分成单词，然后unlist函数将结果处理成字符向量，最后返回键值组合，reduce函数计算各自子任务计数的总和并返回每个单词出现次数的和。
mapreduce函数提交任务，需要4个输入参数，HDFS输入路径、HDFS输出路径，map函数和reduce函数。
# 12.7 比较R MapReduce函数和标准R程序的性能差别
```
rmr.options(backend='local')
a.time <- proc.time()
small.ints2 <- 1:100000
result.normal <- sapply(small.ints2, function(x) x^2)
proc.time() -a.time
   user  system elapsed 
  0.163   0.007   0.176
b.time <-  proc.time()
small.ints <- to.dfs(1:100000)
result <- mapreduce(input=small.ints, map=function(k,v) cbind(v,v^2))
proc.time()-b.time
   user  system elapsed 
  0.931   0.083   1.022
```
时间差了10倍呢，得到了一个计算时间的方法。这是本地模式运行，所以速度快了点呢，如果分布模式下，要花费几分钟以上了。
可以发现，任务不大的情况下，MapReduce方法要完成几十秒的任务也需要几分钟，原因是需要花费一定时间用于启动系统服务、协调不同进程间的任务，从每个节点读取数据。因此，如果我们可以把数据全部放到内存中，就应该采用标准R程序来解决问题，如果数据太大，才可以选择MapReduce，否则应该是“大炮打苍蝇”了吧！
# 12.8 测试和调试rmr2程序
```
rmr.options(backend='local')
b.time <-  proc.time()
small.ints <- to.dfs(1:100000)
result <- mapreduce(input=small.ints, map=function(k,v) cbind(v,v^2))
proc.time()-b.time
 result <- mapreduce(to.dfs(1), map=function(k,v) rmr.str(v))
Dotted pair list of 14
 $ : language mapreduce(to.dfs(1), map = function(k, v) rmr.str(v))
 $ : language mr(map = map, reduce = reduce, combine = combine, vectorized.reduce, in.folder = if (is.list(input)) {     lapply| __truncated__ ...
 $ : language c.keyval(do.call(c, lapply(in.folder, function(fname) {     kv = get.data(fname) ...
 $ : language do.call(c, lapply(in.folder, function(fname) {     kv = get.data(fname) ...
 $ : language lapply(in.folder, function(fname) {     kv = get.data(fname) ...
 $ : language FUN(X[[i]], ...)
 $ : language unname(tapply(1:lkv, ceiling((1:lkv)/(lkv/(object.size(kv)/10^6))), function(r) {     kvr = slice.keyval(kv, r) ...
 $ : language tapply(1:lkv, ceiling((1:lkv)/(lkv/(object.size(kv)/10^6))), function(r) {     kvr = slice.keyval(kv, r) ...
 $ : language lapply(X = ans[index], FUN = FUN, ...)
 $ : language FUN(X[[i]], ...)
 $ : language as.keyval(map(keys(kvr), values(kvr)))
 $ : language is.keyval(x)
 $ : language map(keys(kvr), values(kvr))
 $ : language rmr.str(v)
  ..- attr(*, "srcref")= 'srcref' int [1:8] 1 36 1 59 36 59 1 1
  .. ..- attr(*, "srcfile")=Classes 'srcfilecopy', 'srcfile' <environment: 0x7fe5cde38970> 
v
 num 1
```
这是本地模式运行，所以速度快了点呢，如果分布模式下，要花费几分钟以上了。
# 12.10 使用plyrmr处理数据
rmr2包写mapreduce程序已经相比原生简单多了，但相对一个非程序员难度依然很大，plyrmr包是MapReduce的较高抽象。
```
 yum install libxml2-devel curl-devel -y
conda install -c r r-pryr
install.packages(c("RCurl","httr"), dependencies=TRUE)
install.packages(c("R.methodsS3","hydroPSO"), dependencies=TRUE)
```
发现这个包的安装巨难，怎么尝试都没成功呢，就到这了。
# 12.11 RHadoop中实施机器学习
```
library(MASS)
data(cats)
X <- matrix(cats$Bwt)
Y <- matrix(cats$Hwt)
model <- lm(Y ~ X)
summary(model)
Call:
lm(formula = Y ~ X)

Residuals:
    Min      1Q  Median      3Q     Max 
-3.5694 -0.9634 -0.0921  1.0426  5.1238 

Coefficients:
            Estimate Std. Error t value Pr(>|t|)    
(Intercept)  -0.3567     0.6923  -0.515    0.607    
X             4.0341     0.2503  16.119   <2e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 1.452 on 142 degrees of freedom
Multiple R-squared:  0.6466,    Adjusted R-squared:  0.6441 
F-statistic: 259.8 on 1 and 142 DF,  p-value: < 2.2e-16
library(MASS)
data(cats)
X <- matrix(cats$Bwt)
Y <- matrix(cats$Hwt)
model <- lm(Y ~ X)
summary(model)
library(rmr2)
rmr.options(backend='local')
X <- matrix(cats$Bwt)
X.index <- to.dfs(cbind(1:nrow(X), X))
Y <- as.matrix(cats$Hwt)
Sum <- function(., YY) keyval(1, list(Reduce('+', YY)))
XtX <- values(
  from.dfs(
    mapreduce(
      input <- X.index,
      map <- function(., Xi){
        Xi <- Xi[,-1]
        keyval(1,list(t(Xi) %*% Xi))},
      reduce <- Sum, combine =TRUE)))[[1]]
XtY <- values(
  from.dfs(
    mapreduce(
      input <- X.index,
      map <- function(., Xi){
        Yi <- Y[Xi[,1],]
        Xi <- Xi[,-1]
        keyval(1,list(t(Xi) %*% Yi))},
      reduce <- Sum, combine =TRUE)))[[1]]
solve(XtX, XtY)
```
![cats数据集的线性回归图](https://upload-images.jianshu.io/upload_images/6644753-89a0b5ce6e251ce3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里依然报错，没有得出结果，不过，根据书中的结果，MapReduce方法的系数据比lm模型得到的更不准确。。。，而且差距还是有点的，3.907113 Vs 4.0341
但是如果数据集大致内存无法放下，就无其他选择了。
后面内容就省略了，awz的云应该暂时用不到。
