[返回](../index.md)

# TSDB 数据存储流程及数据压缩过程
![avatar](../images/tsdb_storage.png)

### ScrapeManager 管理targets, 每一个target 生成一个 ScrapePool.

关键协程及关键函数
```
- prometheus/main.go

//生成 scrapeManager 传入fanoutStorage
scrapeManager = scrape.NewManager(log.With(logger, "component", "scrape manager"), fanoutStorage)


- scrape/manager.go 

// 每一个 targetSet 生成一个scrapeLoop, m.append 即为传入的fanoutStorage
    newScrapePool(scrapeConfig, m.append, m.jitterSeed, log.With(m.logger, "scrape_pool", setName)

- scrape/scrape.go
    同步所有的scrapePool
    go func(sp *scrapePool, groups []*targetgroup.Group) {
        sp.Sync(groups)
        wg.Done()
    }(m.scrapePools[setName], groups)
    

    每一个target运行一个协程执行进行抓取动作
    go l.run(interval, timeout, nil)
    
// appender 追加并提交抓取到的数据    
1.    sl.scrapeAndReport(interval, timeout, last, scrapeTime, errc)

2.    app := sl.appender(sl.parentCtx)

3.    sl.append(app, b, contentType, appendTime)
    
4.    app.Append(ref, lset, t, v)

5.    app.Commit()
```

### HeadAppender 追加数据到Head中。
![avatar](../images/chunk.png)
- 通过label生成唯一的ID(metric name 也是label的一部分，以{"\_\_name\_\_"= "$name"}的形式存储)，如果ID存在，追加到series的头部。如果ID不存在， 生成新的series。
- commit 之前追加写入数据到wal 中。


### Head 存储流程
![avatar](../images/head_block.png)
- 每隔两小时截取一个新的block后保存到数据库中
- 截取header的同时删除wal
- 每隔一分钟检测过期数据并删除， 检测块状态， 如果有损坏则退出。

### Compact
- plan 挑选出能够被压缩的块
- 块压缩合并块 meta.json
- 合并块后生成新的 block，生成新的uid 
