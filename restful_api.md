## Restful API

Elasticsearch使用restful的架构模型，所以我们能很方便的与其交互。关于REST的详细介绍，请看[这里](http://en.wikipedia.org/wiki/Representational_state_transfer)。

后续，会使用[httpie](https://github.com/jakubroztocil/httpie)这个工具来与Elasticsearch进行交互。

下面的例子中，我们假设index为blog，而document type为article。

## Create

```
http POST :9200/blog/article/1 title="hello elasticsearch" tags:='["elasticsearch"]'

HTTP/1.1 201 Created
Content-Length: 73
Content-Type: application/json; charset=UTF-8

{
    "_id": "1", 
    "_index": "blog", 
    "_type": "article", 
    "_version": 1, 
    "created": true
}
```

我们也可以让Elasticsearch帮我们生成唯一id

```
http POST :9200/blog/article title="hello elasticsearch" tags:='["elasticsearch"]'

HTTP/1.1 201 Created
Content-Length: 92
Content-Type: application/json; charset=UTF-8

{
    "_id": "AUriBJaUiypP6SwOwQkt", 
    "_index": "blog", 
    "_type": "article", 
    "_version": 1, 
    "created": true
}

```

譬如上面的**AUriBJaUiypP6SwOwQkt**。

## Get

```
http GET :9200/blog/article/1

HTTP/1.1 200 OK
Content-Length: 141
Content-Type: application/json; charset=UTF-8

{
    "_id": "1", 
    "_index": "blog", 
    "_source": {
        "tags": [
            "elasticsearch"
        ], 
        "title": "hello elasticsearch"
    }, 
    "_type": "article", 
    "_version": 1, 
    "found": true
}
```

## Update

### Whole update

```
http PUT :9200/blog/article/1 title="hello elasticsearch" tags:='["elasticsearch", "hello"]'

HTTP/1.1 200 OK
Content-Length: 74
Content-Type: application/json; charset=UTF-8

{
    "_id": "1", 
    "_index": "blog", 
    "_type": "article", 
    "_version": 2, 
    "created": false
}

```

### Partial update

```
http POST :9200/blog/article/1/_update script="ctx._source.tags+=tag" params:='{"tag":"abc"}'
HTTP/1.1 200 OK
Content-Length: 58
Content-Type: application/json; charset=UTF-8

{
    "_id": "1", 
    "_index": "blog", 
    "_type": "article", 
    "_version": 3
}
```


## Delete

```
http DELETE :9200/blog/article/1
HTTP/1.1 200 OK
Content-Length: 71
Content-Type: application/json; charset=UTF-8

{
    "_id": "1", 
    "_index": "blog", 
    "_type": "article", 
    "_version": 4, 
    "found": true
}

http HEAD :9200/blog/article/1
HTTP/1.1 404 Not Found
Content-Length: 0
Content-Type: text/plain; charset=UTF-8
```

## Exists

Exists:

```
http HEAD :9200/blog/article/1
HTTP/1.1 200 OK
Content-Length: 0
Content-Type: text/plain; charset=UTF-8
```

Not exists:

```
http HEAD :9200/blog/article/2
HTTP/1.1 404 Not Found
Content-Length: 0
Content-Type: text/plain; charset=UTF-8
```


