## FileX

FileX 是文件 ETL 工具. 目前已支持 UpyunStore/HDFS/Http 等数据源之间的同步转化.

## Features

+ Filex 通过封装的抽象文件接口和文件转换接口, 理论上能够支持任意数据源之间的数据同步工作. 同时 Filex 支持插件, 能够方便的扩展新的数据源.
+ 使用 `logrus` 实现了 traceID.

## FileX 详细介绍 

### 一. FileX 2.0 概览

![filex1](./pictures/filex1.png)

`IFileSrc` 是输入数据源的文件接口, ITransform 是文件转换的接口, IFileDst 是输出数据源的文件接口. `IFileSrc` 和 `IFileDst` 仅有业务逻辑上的区别, 它们使用的文件接口是一样的.

### 二. 框架设计

FileX 的实现是 OOP 风格的, 如果采用 Java 等对 OOP 支持良好的语言, 代码能更精简易懂.

![filex2](./pictures/filex2.png)

**顶层业务接口**

JobKkmh, JobGifshowLocal, JobGifshowV3 是最上层的业务逻辑. 它们通过继承并重写 `StandardFusion` 部分方法实现业务需求.

**StandardFusion**

StandardFusion 是中间层, 依赖于 `IFile` / `ITransform` 等抽象接口, 实现了数量众多的通用方法供上层业务挑选. 它像一个大的积木桶, 桶里都是按照单一职责类的原则写的小模块.

**IFile**

IFile 是抽象文件系统. 目前 Http, Hdfs 和 UpyunStore(又拍云存储) 三种数据源实现了该接口. IFile 的作用是抹平不同数据源之间的差异, 让 StandardFusion 可以无感知的调用它们.

`Upload` 的参数是 src 是接口 dst 字符串, 而 `Download` 的参数是 src 是字符串 dst 是接口. 为什么正好反过来了? 

因为上传时 dst 是已知的并且 dst 肯定能用字符串这种类似于 URI (资源定位符) 的形式描述出来,  src 设计为接口是为了最大限度的保证适配的灵活性, 比如打开的文件或者 response.body 等都可以适配. Download 设计为相反同理.

**ITransform**

ITransform 对转化进行了抽象. 目前本地转化 Local 和 Flink 已支持该接口.

**ITask**

这里要区分 Job 和 Task 的概念. Job 是一个完整的文件 ETL 处理任务, Task 是 Job 中的一个步骤. 一个 Job 是由 n 个 Task 组合构成一个 pipeline.

Task 之间传递数据使用 `Context`, 这个 Context 的实现非常简单就是一个 `Map` , key 是字符串 value 是 `interface{}`. 设计 Context 的目的是在下一个版本把任务的处理做成分布式的, 不同机器之间同步数据使用序列化的 `Context`. 这样就实现了一个非常简单的 map reduce.

### 三. 参考列表

FileX 从以下项目中汲取了思路和灵感.

+ 项目名称, 文档和最初的思路. => DataX
+ ITask 的 pipeline 思想. => 之前的一个内部框架
+ IFile 的接口设计 => "github.com/colinmarc/hdfs/v2", "github.com/upyun/go-sdk/v3/upyun"
+ Context 的思想和代码实现 => Gin

### 四. TODO

+ 参考 `gcelery` 项目, 将 Filex 实现分布式作业.
+ 结合 `gleam` 项目, 升级为一个分布式通用型的计算框架.

## QA 环节

**1. 你这个任务是不是分了几个步骤: 文件读取阶段是 IFile, 转换阶段是 ITransform, 输出阶段是 ITask?**

不是. 你显然是照着 "二 框架设计" 中的图去思考的, 这个图是 "类图" 不是 "时序图", runtime 的过程更接近 "一. FileX 2.0 概览" 中的图, 就是非常简单的输入, 转换和输出.

`ITask` 的作用是把整个任务组织起来. 

(接下来, 结合相关代码进行说明.)

**1.1 为什么你的任务不分几个不同阶段, 然后可以支持最上层的不同 Job(JobKkmh, JobGifshowLocal ...)**

每个顶层的业务逻辑都继承父类 StandardFusion 然后做针对性的修改, 这样就避免了在 StandardFusion 中使用大量的 `if` 或 `switch` 之类的判断去断定当前运行的哪个任务. `StandardFusion` 作为业务逻辑的中间层, 它 "看不见"(无法感知当前是哪个任务).

即便 StandardFusion 在不用感知顶层任务的情况下也成功区分了不同情况, 这必然导致函数内部的实现复杂化, 不太符合单一职责原则. 使用继承的设计, 就是为了想让每个小模块的代码更精简. 

**2. 为什么在 IFile 的设计中要区分 Download 或 Upload, 不能把文件统统抽象成 open 和 write 两个操作吗? (也就是把本地文件操作系统看做是标准接口, 其他的数据源都来实现这套接口. )比如无论是 http response 的 body 还是 file 读取的内容, 都实现了 io.reader 接口, 你都可以把它看做是一个东西, 进行接下来的操作.**

我有过类似的想法, 也曾实现过. 但是效果并不好. 省略 `Download` 和 `Upload` 直接使用 `Open` 确实在接口设计上更简洁一些.  

但是本地文件系统和远程文件系统是有本质区别的:

+ 本地文件系统默认是可靠的, 比如写入失败后你不会重试, 这一般是盘坏了. 但是远程文件系统写入本质上是 HTTP 或 TCP 传输, 默认是不可靠的, 写入失败了重试是必要操作.
+ 本地文件系统默认是速度最快的. 这个快是和远程文件系统比较起来. 如果你把远程文件系统与本地文件系统混淆, 都 open 之后就去进行接下来的 ETL 步骤. 本地文件毫无问题, 但是如果是远程文件系统的话, 如果你处理的比较慢超时了, 远程的服务器就会把这个响应终止掉. 所以在文件大时, 比如我处理的都是上百G的单个文件, 肯定先落盘再处理比较合适.
+ 远程文件系统由于不一定和接下来的处理程序在一台机器上, 上传下载这些步骤是客观存在的. 如果你承认我说的第2点比较有道理, 那么当你把远程和本地混淆起来后, 就会出现一个临时文件的问题. 临时文件这个问题在本地文件系统是不存在的(你不用像远程那样先落盘再操作, 当然就没有临时文件了).

基于以上理由, 我把本地文件系统和远程文件系统区分开了. 上面的接口 `IFile` 实际上指的是远程文件系统.

## 附录

### Plantuml 源码

[filex2.png](./plantuml/filex2.md)



