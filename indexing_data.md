## Creating indices

Elasticsearch会自动创建index，譬如

```
http POST :9200/blog/article/1 title="hello world"
```

如果blog这个index刚开始不存在，Elasticsearch会自动创建。但我们也能够通过如下方式手动创建:

```
http PUT :9200/blog

HTTP/1.1 200 OK
Content-Length: 21
Content-Type: application/json; charset=UTF-8

{
    "acknowledged": true
}
```

创建的时候指定一些配置

```
http PUT :9200/blog settings:='{"number_of_shards":1, "number_of_replicas": 2}'
HTTP/1.1 200 OK
Content-Length: 21
Content-Type: application/json; charset=UTF-8

{
    "acknowledged": true
}

```

上面我们创建了一个blog索引，使用一个shard，两个replicas。

```
http GET :9200/blog

HTTP/1.1 200 OK
Content-Length: 191
Content-Type: application/json; charset=UTF-8

{
    "blog": {
        "mappings": {}, 
        "settings": {
            "index": {
                "creation_date": "1421133862199", 
                "number_of_replicas": "2", 
                "number_of_shards": "1", 
                "uuid": "Vf0y-AZ-Q8uU26u5ptchkQ", 
                "version": {
                    "created": "1040299"
                }
            }
        }
    }
}
```

我们也能很方便的删除index

```
http DELETE :9200/blog

HTTP/1.1 200 OK
Content-Length: 21
Content-Type: application/json; charset=UTF-8

{
    "acknowledged": true
}

```

## Mappings

Schema Mappings可以认为是关系型数据库中的schema，它会定义index的相关结构。

下面以一个posts index为例，一个blog post，包含如下结构:

+ Unique identifier
+ Name
+ Publication date
+ Contents

对应的mapping文件, posts.json:

```
{
    "mappings": {
        "post": {
            "properties": {
                "id" : {"type":"long", "store":"yes"},
                "name" : {"type":"string", "store":"yes", "index":"analyzed"},
                "published" : {"type":"date", "store":"yes"},
                "contents" : {"type":"long", "store":"no", "index":"analyzed"}
            }
        }
    }
}
```

```
http POST :9200/posts < posts.json 

HTTP/1.1 200 OK
Content-Length: 21
Content-Type: application/json; charset=UTF-8

{
    "acknowledged": true
}

```

```
http GET :9200/posts 

HTTP/1.1 200 OK
Content-Length: 402
Content-Type: application/json; charset=UTF-8

{
    "posts": {
        "mappings": {
            "post": {
                "properties": {
                    "contents": {
                        "index": "analyzed", 
                        "type": "long"
                    }, 
                    "id": {
                        "store": true, 
                        "type": "long"
                    }, 
                    "name": {
                        "store": true, 
                        "type": "string"
                    }, 
                    "published": {
                        "format": "dateOptionalTime", 
                        "store": true, 
                        "type": "date"
                    }
                }
            }
        },
    ...
}
```

## Core types

+ String
+ Number
+ Date
+ Boolean
+ Binary

### Common attributes

+ index_name
+ index：analyzed，这个field就会被索引并且能够搜索，no，不能被索引，如果是string，则还有not_analyzed，表明能被索引，不能被analyzed，只能完全匹配
+ store：yes，原始值被写入index，no，不写入，默认为no
+ boost：默认为1，值越高，表明这个field越重要
+ null_value
+ copy_to
+ include_in_all

### String

+ term_vector
+ omit_norms
+ analyzer：定义indexing和searching使用的analyzer
+ index_analyzer
+ serach_analyzer
+ norms.enabled
+ norms.loading
+ position_offset_gap
+ index_options
+ ignore_above

### Number

包括byte, short, integer, long, float, double

+ precision_step
+ ignore_malformed

### Boolean

true or false

### Binary

在index使用base64编码存储，只有index_name属性

### Date

+ format：日期格式，如"YYYY-mm-dd"
+ precision_step
+ ignore_malformed


### Multi fields

```
"name" : {
    "type" : "string",
    "fields" : {
        "facet" : {"type" : "string", "index":"not_analyzed"}
    }
}
```

### IP Address

```
"address" : {"type" : "ip", "store" : "yes"}
```

### token_count

```
"address_count" : {"type" : "token_count", "store" : "yes"}
```

## Analyzers

使用[ik](https://github.com/medcl/elasticsearch-analysis-ik)进行中文分词处理