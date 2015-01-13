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

