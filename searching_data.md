## 数据准备

mapping json

```
http POST :9200/library

http PUT :9200/library/book/_mapping < data/book_mapping.json
```

bulk data

```
http POS :9200/_bulk < data/book_document.json 
```

## Simple query

```
http GET :9200/library/book/_search query:='{"query_string":{"query":"title:crime"}}'

HTTP/1.1 200 OK
Content-Length: 446
Content-Type: application/json; charset=UTF-8

{
    "_shards": {
        "failed": 0, 
        "successful": 5, 
        "total": 5
    }, 
    "hits": {
        "hits": [
            {
                "_id": "4", 
                "_index": "library", 
                "_score": 0.15342641, 
                "_source": {
                    "author": "Fyodor Dostoevsky", 
                    "available": true, 
                    "characters": [
                        "Raskolnikov", 
                        "Sofia Semyonovna Marmeladova"
                    ], 
                    "copies": 0, 
                    "otitle": "Преступлéние и наказáние", 
                    "tags": [], 
                    "title": "Crime and Punishment", 
                    "year": 1886
                }, 
                "_type": "book"
            }
        ], 
        "max_score": 0.15342641, 
        "total": 1
    }, 
    "timed_out": false, 
    "took": 2
}

```

## Paging and size

from and size

```
http GET :9200/library/book/_search query:='{"query_string":{"query":"title:crime"}}' from=20 size=9

HTTP/1.1 200 OK
Content-Length: 128
Content-Type: application/json; charset=UTF-8

{
    "_shards": {
        "failed": 0, 
        "successful": 5, 
        "total": 5
    }, 
    "hits": {
        "hits": [], 
        "max_score": 0.15342641, 
        "total": 1
    }, 
    "timed_out": false, 
    "took": 1
}
```

## 返回版本号

version

```
http GET :9200/library/book/_search query:='{"query_string":{"query":"title:crime"}}' version:='true'
```

## 限制最低score

min_score

```
http GET :9200/library/book/_search query:='{"query_string":{"query":"title:crime"}}' min_score:='0.75'
```

## 返回指定field

fields

```
http GET :9200/library/book/_search query:='{"query_string":{"query":"title:crime"}}' fields:='["title", "year"]'
```

## Partial field

partial_fields，有include和exclude两个属性

```
http GET :9200/library/book/_search query:='{"query_string":{"query":"title:crime"}}' partial_fields:='{"partial1":{"include":["titl*"], "exclude":["chara*"]}}'

HTTP/1.1 200 OK
Content-Length: 250
Content-Type: application/json; charset=UTF-8

{
    "_shards": {
        "failed": 0, 
        "successful": 5, 
        "total": 5
    }, 
    "hits": {
        "hits": [
            {
                "_id": "4", 
                "_index": "library", 
                "_score": 0.15342641, 
                "_type": "book", 
                "fields": {
                    "partial1": [
                        {
                            "title": "Crime and Punishment"
                        }
                    ]
                }
            }
        ], 
        "max_score": 0.15342641, 
        "total": 1
    }, 
    "timed_out": false, 
    "took": 2
}
```

## Script fields

script_fields

```
http GET :9200/library/book/_search query:='{"query_string":{"query":"title:crime"}}' script_fields:='{"correctYear":{"script":"_source.year - 1800"}}'

HTTP/1.1 200 OK
Content-Length: 223
Content-Type: application/json; charset=UTF-8

{
    "_shards": {
        "failed": 0, 
        "successful": 5, 
        "total": 5
    }, 
    "hits": {
        "hits": [
            {
                "_id": "4", 
                "_index": "library", 
                "_score": 0.15342641, 
                "_type": "book", 
                "fields": {
                    "correctYear": [
                        86
                    ]
                }
            }
        ], 
        "max_score": 0.15342641, 
        "total": 1
    }, 
    "timed_out": false, 
    "took": 4
}
```

给Script传递参数

```
http GET :9200/library/book/_search query:='{"query_string":{"query":"title:crime"}}' script_fields:='{"correctYear":{"script":"_source.year - paramYear", "param":{"paramYear":1800}}}'
```

## Search Types

+ query_then_fetch，默认的，首先在所有相关的shard上面执行语句，获取相关相关信息用以sort和rank documents，然后在从满足条件的shard里面查询数据。
+ query_and_fetch，query在所有的shard上面执行并返回结果
+ dfs_query_and_fetch
+ dfs_query_then_fetch
+ count，只是返回满足条件的文档数量
+ scan

通过参数search_type指定

```
http GET :9200/library/book/_search query:='{"query_string":{"query":"title:crime"}}' search_type==query_and_fetch
```

## Term Query

```
http GET :9200/library/book/_search query:='{"term":{"title":"crime"}}'
```

Term因为不可分割，只能是全匹配，并且不会被analyzed

## Terms Query

```
http GET :9200/library/book/_search query:='{"terms":{"tags":["novel", "book"], "minimum_match":1}}' 
```

minimal_match等于1，只需要一个term匹配就可以了，如果为2，就是两个都需要匹配

## match_all Query

返回所有的documents

```
http GET :9200/library/book/_search query:='{"match_all":{}}'
```

## match query

```
http GET :9200/library/book/_search query:='{"match":{"title":"crime and punishment"}}'
```

## Boolean match query

+ operator: and/or
+ analyzer
+ fuzziness
+ prefix_length
+ max_expansions
+ zero_terms_query
+ cutoff_frequency

```
http GET :9200/library/book/_search query:='{"match":{"title":{"query":"crime and punishment", "operator":"and"}}}'
```

## match_phrase query

+ slop
+ analyzer

```
http GET :9200/library/book/_search query:='{"match_phrase":{"title":{"query":"crime punishment", "slop":1}}}'
```

## match_phrase_prefix query

## multi_match query

可以指定多个field进行查询

```
http GET :9200/library/book/_search query:='{"multi_match":{"query":"crime punishment","fields":["title", "otitle"]}}'
```

## query_string query

支持Lucene语法

```
http GET :9200/library/book/_search query:='{"query_string":{"query":"title:crime^10 +title:punishment -otitle:cat +author:(+Fyodor +dostoevsky)", "default_field":"title"}}'
```

+ query
+ default_field
+ default_operator
+ analyzer
+ allow_leading_wildcard
+ lowercase_expand_terms
+ enable_position_increments
+ fuzzy_max_expansions
+ fuzzy_prefix_length
+ fuzzy_min_sim
+ phrase_slop
+ boost
+ analyze_wildcard
+ auto_generate_phrase_queries
+ minimum_should_match
+ lenient

mutli fields

```
http GET :9200/library/book/_search query:='{"query_string":{"query":"title:crime^10 +title:punishment -otitle:cat +author:(+Fyodor +dostoevsky)", "default_field":"title", "use_dis_max": true}}'
```

## simple_query_string query

不同于query_string，对于错误query，不会抛出异常，只会忽略错误的部分

## identifiers query

只针对id

```
http GET :9200/library/book/_search query:='{"ids":{"values":["1", "10"]}}'
```

限定document type

```
http GET :9200/library/book/_search query:='{"ids":{"values":["1", "10"], "type": "book"}}'
```

## prefix query

```
http GET :9200/library/book/_search query:='{"prefix":{"title":"cri"}}'
```

## fuzzy_like_this query
## fuzzy_like_this_field query
## fuzzy query
## wildcard query
## more_like_this query
## more_like_this_field query

## range query

+ gte
+ gt
+ lte
+ lt

```
http GET :9200/library/book/_search query:='{"range":{"year":{"gte":1700, "lte":1900}}}'
```

## dismax query
## regular expression query

## bool query

+ should
+ must
+ must_not

## boosting query
## constant_score query
## indices query

## Filters

post_filter

```
http GET :9200/library/book/_search query:='{"match":{"title" : "Catch"}}' post_filter:='{"term":{"year":1961}}'
```

filtered

```
http GET :9200/library/book/_search query:='{"filtered":{"query":{"match":{"title": "Catch"}}, "filter":{"term":{"year":1961}}}}'
```

## Filter types

+ range 
+ exists 
+ missing 
+ script
+ type
+ limit
+ identifiers

## Highlighting

```
http GET :9200/library/book/_search query:='{"term":{"title":"crime"}}' highlight:='{"fields":{"title":{}}}'
```

## Sort

asc or desc

如果想让某个field排序，一定要在mapping里面指定，否则都会按照score来排序的
