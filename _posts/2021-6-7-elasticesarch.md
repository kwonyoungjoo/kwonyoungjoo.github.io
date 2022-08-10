---
layout: post
title:  엘라스틱서치 검색 쿼리 작성하기
---
<p>엘락스트 검색 쿼리 예제 입니다. 자주 사용하지 않다 보니 필요할떄 마다 잊어버려서 예제 작성을 해두었습니다.</p>

<p>최대, 최소, 마지막 값 구하기</p>

```python
query = {
        "bool": {
            "must": [
                {"range": {
                    "@timestamp": {
                        "time_zone": "+09:00",
                        "gte": gte,
                        "lt": lte
                    }
                }
                },
                {"term": {"status": "ENABLE"}}
            ],
        }
    }

    aggs = {
        "unique_id": {
            "terms": {
                "field": "stockId",
                "size": 1000
            },
            "aggs": {
                "max_usd": {
                    "top_hits": {
                        "size": 1,
                        "_source": ["@timestamp", "field1", "field2", "field3"],
                        "sort": [{
                            "lastUsd": {"order": "desc"}
                        }
                        ]
                    }
                },
                "min_usd": {
                    "top_hits": {
                        "size": 1,
                        "_source": ["@timestamp", "field1", "field2", "field3"],
                        "sort": [{
                            "lastUsd": {"order": "asc"}
                        }
                        ]
                    }
                },
                "last_usd": {
                    "top_hits": {
                        "size": 1,
                        "_source": ["@timestamp", "field1", "field2", "field3"],
                        "sort": [{
                            "@timestamp": {"order": "desc"}
                        }
                        ]
                    }
                }
            }
        }
    }

    res = es.search(index=get_index_name(conf), doc_type='_doc', size=_FETCH_SIZE, query=query, aggs=aggs)

    data = res['aggregations']

    return data['unique_id']['buckets']
```

```python
es_hosts = "host1,host2"
es = Elasticsearch(hosts=es_hosts, http_auth=('id', 'passwd'), 
									timeout=10, max_retries=0,
                   retry_on_timeout=False)

query = {
"bool": { 
	"must": [
		{"range": {
			"@timestamp": {
					"timezone": "+09:00", 
					"gte": "2022-01-01T00:00:00",
					"lte": "2022-02-01T00:00:00"
					}
			} 
		] 
	}, 
    aggs = {"click_cnt" :{
				 "terms": { 
					"field": "field_name"
				}
			}
		}
    }

res = es.search(index="index_name", doc_type="_doc", size=0, query=query, aggs=aggs) 
```

<p>collections.Counter</p>

```python
from collections import Counter

Counter('test').most_commont()
[('t', 2), ('e', 1), ('s', 1)]

Counter('test) 
{'t': 2, 'e': 1, 's': 1}
```

<p>List.sort()</p>

```python
list_str = ['3', '2', '1', '4', '9']
list_str.sort(reverse=False)
list_str 
['1', '2', '3', '4', '9']

```

<p>sorted</p>

```python
sorted({3: 'D', 2: 'B', 5: 'B', 4: 'E', 1: 'A'})
[1,2,3,4,5]

students = [
        ('홍길동', 3.9, 2016303),
        ('김철수', 3.0, 2016302),
        ('최자영', 4.3, 2016301),
]
sorted(students, key=lambda student: student[2], reverse=False)
```

<p>regex</p>

```python
import re
p = re.compile('([1-9])([a-z])')
s = '1abdkjk3cd' 
p.findall(s)
[('1', 'a'), ('3', 'c')]

p = re.compile('([1-9])([a-z]*)')
p.findall(s)
[('1', 'abdkjk'), ('3', 'cd')]
```

