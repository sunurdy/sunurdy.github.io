---
layout:     post
title:      Geo Hash
category: blog
description: 关于Geo地理数据分组聚合的算法
---​
## GEO
上周世界各地爆发wanna-cry病毒，我们也是周末加班，做出中国区域感染分布图。

匆匆做完后发现，数据点太多，前端渲染需要很长时间，卡顿感强烈。于是着手优化，写出两种方案。

### 1.geohash
[geohash](https://en.wikipedia.org/wiki/Geohash)就是将经纬度hash成为一个字符串，将地图分为一块一块的格子。截取从首字母开始任意长度的字符串都可以代表一个格子，截取的字符串越长，格子划分越小。
两个坐标对应的geohash如果前几位字母相同，代表这两个坐标在那个level的格子中。

这里可以看到划分的级别与距离对应[geohash_in_es](https://www.elastic.co/guide/en/elasticsearch/guide/current/geohashes.html)

首先便可以使用geohash做划分，先确定level，遍历一次数据，就可以将他们分布在那个level下的格子中，每个格子只取一个数据，这样就过滤了很多数据点。
```python
class GeoHashMap(object):
    d = dict()

    def __init__(self, level=5):
        self.level = level

    def set(self, item):
        location = item.get('location')
        p_lat, p_lng = location.split(',')
        p_lat = float(p_lat.strip())
        p_lng = float(p_lng.strip())
        hash = geohash.encode(p_lat, p_lng)
        self.d[hash[:self.level]] = item

    def result(self):
        return self.d.keys()
```
这个可以做到过滤数据，但是我们不能指定格子的精度。只能按照他的固定范围。SO

### 2.自己划分格子
首先指定精度，比如10km。根据查询的范围bbox，计算长宽。于是得到了整个范围划分的格子数height*width。
而且我们得到了每一个步长对应的delta经纬度。
接下来就好办了，对每一个数据坐标，我们可以知道它落在哪个格子里面，如此便哈希。
最后每个格子只留一个点就行了。
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from __future__ import division

import math
from functools import partial

def distance(origin, destination, direction=None):
    lon1, lat1 = origin
    lon2, lat2 = destination
    radius = 6371  # km

    dlat = math.radians(lat2 - lat1)
    dlon = math.radians(lon2 - lon1)
    if direction == 'lat':
        dlon = 0
    elif direction == 'lng':
        dlat = 0

    a = math.sin(dlat / 2) * math.sin(dlat / 2) + math.cos(math.radians(lat1)) * math.cos(
            math.radians(lat2)) * math.sin(dlon / 2) * math.sin(dlon / 2)
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))
    d = radius * c
    return d

distance_lat = partial(distance, direction='lat')
distance_lng = partial(distance, direction='lng')

class HashPointMap(object):
    # height=delta_lat --> y
    # width =delta lng  --> x
    d = dict()

    def __init__(self, bbox, delta_km=10):
        # bbox = ((lon1, lat1), (lon2, lat2))

        dlat_km = distance_lat(bbox[0], bbox[1])
        dlng_km = distance_lng(bbox[0], bbox[1])
        # min lat lng
        self.lat_min = bbox[0][1] if bbox[0][1] < bbox[1][1] else bbox[1][1]
        self.lng_min = bbox[0][0] if bbox[0][0] < bbox[1][0] else bbox[1][0]

        # 总共有多少格子
        self.height = int((dlat_km // delta_km) + 1)
        self.width = int((dlng_km // delta_km) + 1)
        # 每个格子的经纬度delta
        self.delta_lng = abs((bbox[0][0] - bbox[1][0]) / self.width)
        self.delta_lat = abs((bbox[0][1] - bbox[1][1]) / self.height)

    def set(self, item):
        location = item.get('location')
        p_lat, p_lng = location.split(',')
        p_lat = float(p_lat.strip())
        p_lng = float(p_lng.strip())
        x, y = self.get_x_y(p_lat, p_lng)
        key = "%s_%s" % (x, y)
        self.d[key] = item

    def get_x_y(self, p_lat, p_lng):
        x = (p_lng - self.lng_min) // self.delta_lng
        y = (p_lat - self.lat_min) // self.delta_lat
        return x, y

    def result(self):
        return self.d.keys()
```
### 测试

```python
if __name__ == '__main__':
    import time
    client = Elasticsearch(hosts=[""])
    s = Search(using=client, index='')
    response = s.scan()
    response = [hit.to_dict() for hit in response]
    print len(response)
    print 50 * '-'

    t = time.time()
    bbox = ((72, 54), (136, 17))
    hashmap = HashPointMap(bbox, delta_km=1)
    for i in response:
        hashmap.set(i)
    print time.time() - t
    res = hashmap.result()
    print len(res)
    
    print 50 * '-'
    
    t = time.time()
    hashmap = GeoHashMap(level=6)
    for i in response:
        hashmap.set(i)
    print time.time() - t
    res = hashmap.result()
    print len(res)
```
结果看来自己的方法更快，geohash计算起来费时了
```
47426
--------------------------------------------------
0.167999982834
13815
--------------------------------------------------
1.21199989319
15666
```


