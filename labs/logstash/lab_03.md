# Lab 03. Discover Logstash status.

A. Node information:
```
$ curl -XGET localhost:9600/_node?pretty
```

B. Deep statistics:
```
$ curl -XGET localhost:9600/_node/stats?pretty
```
