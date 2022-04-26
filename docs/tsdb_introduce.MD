[返回](../index.md)


## TSDB 介绍

### tsdb目录结构
```
 ./data 
├── 01BKGV7JBM69T2G1BGBGM6KB12 

│   └── meta.json
├── 01BKGTZQ1SYQJTR4PB43C8PD98
│   ├── chunks
│   │   └── 000001
│   ├── tombstones
│   ├── index
│   └── meta.json
├── 01BKGTZQ1HHWHV8FBJXW1Y3W0K
│   └── meta.json
├── 01BKGV7JC0RY8A6MACW02A2PJD
│   ├── chunks
│   │   └── 000001
│   ├── tombstones
│   ├── index
│   └── meta.json
├── chunks_head
│   └── 000001
└── wal
    ├── 000000002
    └── checkpoint.00000001
        └── 00000000
```

### data：
数据库文件总目录
<br><br>

### Block：
- 数据存储块，prometheus 数据存储时间分片单元。
- 每一个块中基础保存2小时的数据。
- 小的数据块可合并成大的数据块。
 #### Block-chunks：
- chunks 中包含保存监控数据的文件
- 单个chunk默认为256MB大小（可配置）

### Block-tombstones：
- 墓碑文件，block中部分过期的chunk会放入tombstones 文件中。
- 当block 完全过期，则被删除。
<br><br>
### index文件：
![avatar](../images/index.png)
series 列表：记录当前 block 中有哪些 series，每个 series 有哪些 label 值，有哪些 chunks，每个 chunk 的开始、结束时间；  
倒排索引（posting）：每个 label 值到 series id 列表的倒排；  
倒排索引的 offset table：每个倒排列表的起始 offset；  
toc：index 文件中各部分的起始 offset；  

### meta.json文件：
- json格式文件负责记录block 信息。
- block中sample 数量
- block的父级blokc记录。
- boock 压缩等级记录。
- 每次block的合并会基于meta.json文件来进行合并，合并后meta.json 文件也会合并。


## 启动参数

```
启动参数                         ｜ 默认值   ｜ 含义
--storage.tsdb.path             ｜ /data   |  数据存储路径
--storage.tsdb.retention.time   |  15d     ｜ 样本数据在存储中保存的时间。超过该时间限制的数据就会被删除。
--storage.tsdb.retention.time    ｜ 30d     ｜ 存储过期时间，存储的数据超过这个时间即会被删除

​```