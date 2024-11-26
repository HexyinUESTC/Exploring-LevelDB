# level DB源码解读

# 1 背景

### 1.1 什么是level DB
- level db是谷歌写的Key-Value存储库，可以嵌入到应用当中。是Log-structure Merged-Tree的一种实现

### 1.2 为什么学习level DB
- 我曾经参与过OceanBase的暑期实习，OceanBase所采用的存储架构就是LSM-Tree，可是我当时对LSM-Tree一窍不通，后边只是简单看了几个博客，增加了一些概念性的了解。直到后边秋招腾讯面试深入问了LSM—Tree，那个时候发现我对此知之甚少。于是我便想深入看一下LSM—Tree。
- 之所以选择level DB是因为这个项目开源、经典，可以在和轻松在网上找到很多参考，且代码量少，代码编写规范易于阅读。

## 2 源码解读

### 2.1 代码结构
我们从github上下载源码之后可以看到主要文件如下：
1. /db：这个里边包括了主要的代码文件：用户的方法入口，LSM-Tree中内存的结构，版本信息，日志等等
2. /include：头文件
3. /table：记录了磁盘上文件的组织，迭代器，过滤器等等
4. /util：一些工具类

### 2.2 主要的类
1. Version  
    Version是leveldb某一具体版本，主要记录了包含数据库在某一时刻的所有文件以及元数据的信息。  
    * 记录了压缩相关信息，包括需要压缩的下一文件，需要压缩文件所在的级别，
    * 记录了所有的文件列表
    * 所有的Version会被维护成一个双向链表，所以还有两个指针以及一个指向版本链的指针
    * 向上提供查找以及压缩的辅助接口，以及引用计数的接口【这里不使用智能指针是出于兼容性的考虑】
2. VersionSet  
    维护当前和管理当前数据库中的多个版本，并提供对这些版本的操作和访问



## 2 主要操作
LevelDB对外的主要接口集中在`db_impl.h`中，从这里边的操作深入下去，可窥探leveldb的整体架构以及操作。

### 2.1 DB::Open
1. 创建DbImpl实例




## VersionEdit里边存储的记录是怎么写到磁盘上的？是怎么组织的？
* 校验和：前4个字节，用于验证记录的完整性。
* 长度：第5和第6个字节，表示记录数据的长度。
* 类型：第7个字节，表示记录的类型。
* 记录数据：紧随记录头之后，其长度由长度字段指定。
    - tag，data；tag，data，……，tag，data
## 一个Manifest文件里边有多少个VersionEdit？ 
    有多个VersionEdit，recover通过apply从顶向下依次更新，所以最终内存中的版本一定是在上次关闭之前最新的。