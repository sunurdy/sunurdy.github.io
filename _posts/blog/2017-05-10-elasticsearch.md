---
layout:     post
title:      Elasticsearch
category: blog
description: elasticsearch
---

## Elasticsearch (+Python dsl)

最近有项目需求查询GEO信息，将一个坐标附近10km的信息全部展示出来。于是升级ES到5.0以上，发现和以前2.0版本有一些区别。记录一下ES相关基础知识点。

### INDEX
创建index可以有前缀，设置create_index_action

```
curl -XPUT locahot:9200/create_index_action/realtime_data?pretty -d \
'{
	"settings": {
		"index.refresh_interval": "120s"
	},
	"mappings": {
		"pseudo_realtime_data": {
			"dynamic": false,
			"properties": {
				"content": { "type": "text"},
				"gid": { "type": "keyword"},,
				"lat": { "type": "float"},
				"lng": { "type": "float"},
				"location": { "type": "geo_point"},
				"province":{ "type": "keyword"},
				"save_time": { "type": "date"}
			}
		}
	}
}'
```
所有的字段type在[这里]([https://www.elastic.co/guide/en/elasticsearch/reference/5.3/mapping-types.html)
#### string
以前type都是text，分词靠的是 'index':'not_analyzed'

现在type分为text，keyword keyword为不分词.

文档中text中还可以包含keyword的属性以支持filter操作，写法更加面向对象了

#### geo
用到了[geo-point](https://www.elastic.co/guide/en/elasticsearch/reference/5.3/geo-point.html)
多种写法，我最喜欢的是这样
```
"location": {
	"lat": 22.21791458,
	"lon": 113.55188751
}
```


### 查询
filter是过滤会有缓存，query是按照score搜索。以前版本中filter query使用起来并没有太大区别。

在新版本(>5)中type为text，filter查不出数据，并且无法aggregation

[原生查询语句](https://www.elastic.co/guide/en/elasticsearch/reference/5.3/query-dsl.html)

[Python dsl](http://elasticsearch-dsl.readthedocs.io/en/latest/search_dsl.html)
#### python-es-dsl
Query combination时可以

```
g_list = map(lambda x: Q('term', gid=x), gids)
q = reduce(lambda x, y: x | y, g_list)
s = s.filter(q)
```
也可以
```
s = s.filter('terms', gid=gids)
```
其实filter函数就是query的变形，使用了filter关键字
```
def filter(self, *args, **kwargs):
	return self.query(Bool(filter=[Q(*args, **kwargs)]))
```
时间range 可以直接填写datetime对象
```
start_time = datetime.datetime.fromtimestamp(float(start_time) / 1000)
end_time = datetime.datetime.fromtimestamp(float(end_time) / 1000)
s = s.filter('range', save_time={'gte': start_time, 'lte': end_time})
```
GEO位置查询
```
s = s.filter('geo_distance', distance='10km', location=[lng, lat])
```

## TODO
