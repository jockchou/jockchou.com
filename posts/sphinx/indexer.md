<!--
author: jockchou
date: 2015-08-25
title: coreseek自动化增量索引配置
tags: coreseek, Sphinx，搜索引擎
category: Sphinx
status: publish
summary: 有这么一种常见的情况：整个数据集非常大，以至于难于经常性地重建索引，但是每次新增的记录却相对较少。一个典型的例子是：一个论坛有1000000个已经归档的帖子，但每天只有1000个新帖子。
-->

有这么一种常见的情况：整个数据集非常大，以至于难于经常性地重建索引，但是每次新增的记录却相对较少。一个典型的例子是：一个论坛有1000000个已经归档的帖子，但每天只有1000个新帖子。

在这种情况下可以用所谓的“主索引＋增量索引”（main+delta）模式来实现“近实时”的索引更新。这种方法的基本思路是设置两个数据源和两个索引，对很少更新或根本不更新的数据建立主索引，而对新增文档建立增量索引。在上述例子中，那1000000个已经归档的帖子放在主索引中，而每天新增的1000个帖子则放在增量索引中。增量索引更新的频率可以非常快，而文档可以在出现几分种内就可以被检索到。

确定具体某一文档的分属哪个索引的工作可以自动完成。一个可选的方案是，建立一个计数表，记录将文档集分成两部分的那个文档ID，而每次构建索引时，这个表都会被更新。

## 增量索引配置  ##

```nginx
#定义subject主索引数据源
source pingo_subject_main
{
    type                    = mysql

    sql_host                = 127.0.0.1
    sql_user                = username
    sql_pass                = password
    sql_db                  = dbname
    sql_port                = 3306
    
    sql_query_pre           = SET NAMES utf8
    sql_query_pre           = SET SESSION query_cache_type = OFF
    
    sql_query_pre           = CREATE TABLE IF NOT EXISTS sph_counter(counter_id INTEGER PRIMARY KEY NOT NULL, max_doc_id INTEGER NOT NULL) DEFAULT CHARSET=utf8
    sql_query_pre           = REPLACE INTO sph_counter SELECT 2, MAX(id) FROM subject
    
    sql_query_range         = SELECT MIN(id), MAX(id) FROM subject WHERE id <= (SELECT max_doc_id FROM sph_counter WHERE counter_id = 2)
    sql_range_step          = 1000
    sql_ranged_throttle     = 0
    
    sql_query               = SELECT id, id AS subjectid, title, content, imageUrl, imageUrl2, posterUrl, topicCnt, readCnt, userCnt, orderVal, isActivity, activityTitle, activityUrl, UNIX_TIMESTAMP(addTime) AS addTime, UNIX_TIMESTAMP(updateTime) AS updateTime, UNIX_TIMESTAMP(onlineTime) AS onlineTime, isOfficial, isHot, state FROM subject \
                              WHERE id <= (SELECT max_doc_id FROM sph_counter WHERE counter_id = 2) \
                              AND id >= $start AND id <= $end
    
    sql_attr_uint           = subjectid
    sql_field_string        = title
    sql_attr_string         = content
    sql_attr_string         = imageUrl
    sql_attr_string         = imageUrl2
    sql_attr_string         = posterUrl
    sql_attr_uint           = topicCnt
    sql_attr_uint           = readCnt
    sql_attr_uint           = userCnt
    sql_attr_uint           = orderVal
    sql_attr_uint           = isActivity
    sql_attr_string         = activityTitle
    sql_attr_string         = activityUrl
    sql_attr_timestamp      = addTime
    sql_attr_timestamp      = updateTime
    sql_attr_timestamp      = onlineTime
    sql_attr_uint           = isOfficial
    sql_attr_uint           = isHot
    sql_attr_uint           = state
    
    sql_query_info          = SELECT id, title, content, state FROM subject WHERE id = $id
}

#定义subject增量索引数据源
source pingo_subject_delta : pingo_subject_main
{
    sql_query_pre           = SET NAMES utf8
    
    sql_query_post_index    = UPDATE sph_counter SET max_doc_id = $maxid WHERE counter_id = 2 AND $maxid > 0
    
    sql_query_range         = SELECT MIN(id), MAX(id) FROM subject WHERE id > (SELECT max_doc_id FROM sph_counter WHERE counter_id = 2)
    
    sql_query               = SELECT id, id AS subjectid, title, content, imageUrl, imageUrl2, posterUrl, topicCnt, readCnt, userCnt, orderVal, isActivity, activityTitle, activityUrl, UNIX_TIMESTAMP(addTime) AS addTime, UNIX_TIMESTAMP(updateTime) AS updateTime, UNIX_TIMESTAMP(onlineTime) AS onlineTime, isOfficial, isHot, state FROM subject \
                              WHERE id > (SELECT max_doc_id FROM sph_counter WHERE counter_id = 2) \
                              AND id >= $start AND id <= $end
}


#subject主索引
index subject
{
    source                  = pingo_subject_main
    path                    = /usr/local/coreseek/var/data/subject
    docinfo                 = extern
    charset_dictpath        = /usr/local/mmseg3/etc
    charset_type            = zh_cn.utf-8
    ngram_len               = 0
}

#subject增量索引
index subject_delta
{
    source                  = pingo_subject_delta
    path                    = /usr/local/coreseek/var/data/subject_delta
    docinfo                 = extern
    charset_dictpath        = /usr/local/mmseg3/etc
    charset_type            = zh_cn.utf-8
    ngram_len               = 0
}

```

请注意，上例中我们显示设置了数据源pingo_subject_delta的sql_query_pre选项，覆盖了全局设置。必须显示地覆盖这个选项，否则对delta做索引的时候也会运行那条REPLACE查询，那样会导致delta源中选出的数据为空。可是简单地将delta的sql_query_pre设置成空也不行，因为在继承来的数据源上第一次运行这个指令的时候，继承来的所有值都会被清空，这样编码设置的部分也会丢失。因此需要再次显式调用编码设置查询。

通过以上配置以后，每次更新索引时，都会把数据源当前最大的id存储到sph_counter表中，下次更新索引，只是从这个位置开始，获取新增数据构建索引。



## 构建主索引  ##

主索引在首次使用时构建，以后通过定时任务更新主索引，可以选择在系统压力小的时候更新主索引。
```
/usr/local/coreseek/bin/indexer --rotate --quiet --config /usr/local/coreseek/etc/csft.conf subject 
```

## 构建增量索引 ##

增量索引地更新频率可以比较快一点，视系统的数据写入频率而定。
```
/usr/local/coreseek/bin/indexer --rotate --quiet --config /usr/local/coreseek/etc/csft.conf subject_delta
```

## 合并索引 ##
生成增量索引后，需要把它合并到主索引中，这样数据才能被查询到。可以在增量索引生成后，立即执行合并操作。

```
/usr/local/coreseek/bin/indexer --rotate --quiet --config /usr/local/coreseek/etc/csft.conf --merge subject subject_delta
```