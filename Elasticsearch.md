# ES常用配置

## Kibana可视化界面

**Kibana的版本需要和ES的版本保持一致**

[download](

## 配置跨域

`config/elasticsearch.yml`

```yml
http.cors.enabled: true
http.cors.allow-origin: "*"
```

## 配置占用的内存大小

`config/`jvm.options 默认启动占用`1G`内存

```yml
-Xms512m
```

https://www.elastic.co/cn/downloads/kibana)

