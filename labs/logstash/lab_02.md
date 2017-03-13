# Lab 02. Hello, World!

Configure Logstash:
_/etc/logstash/logstash.yml_
```
config.reload.automatic: true
log.level: verbose
```

Configure Logstash pipeline:
_/etc/logstash/conf.d/trex_hello_world.conf_ 

```
input {
  tcp {
    type => "hello_world"
    port => "10000"
  }
}

filter {
  grok {
    match => { 
      "message" => "Hello, %{WORD:name}" 
    }
  }
}

output {
  stdout {
    codec => rubydebug
  }
  elasticsearch {
    hosts => [ "localhost:9200" ]
    index => "trex_%{type}"
  }
}
```
Save the file.

Stop Logstash and check the config:
```
sudo service logstash stop
sudo bash /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/trex_hello_world.conf --config.test_and_exit --path.settings /etc/logstash/
```

Console output:
```
Configuration OK
```

Start Logstash daemon directly:
```
sudo bash /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/trex_hello_world.conf --config.reload.automatic --path.settings /etc/logstash/
```

Input some data for Logstash:
```
echo 'Hello, Trex!' | nc localhost 10000
```

Logstash daemon console output:
```
{   
    "@timestamp" => 2017-03-13T13:36:37.405Z,
          "port" => 47736,
      "@version" => "1",
          "host" => "0:0:0:0:0:0:0:1",
          "name" => "Trex",
       "message" => "Hello, Trex!",
          "type" => "hello_world"
}
```


Check data in Elasticsearch:
```
curl -XGET localhost:9200/trex_hello_world/_search?pretty
```

Elasticsearch console output:
```
{ 
  "took" : 24,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 1.0,
    "hits" : [ {
      "_index" : "trex_hello_world",
      "_type" : "hello_world",
      "_id" : "AVrHite8S7jrJ-hyMHRi",
      "_score" : 1.0,
      "_source" : {
        "@timestamp" : "2017-03-13T12:00:48.871Z",
        "port" : 49948,
        "@version" : "1",
        "host" : "0:0:0:0:0:0:0:1",
        "name" : "Trex",
        "message" : "Hello, Trex!",
        "type" : "hello_world"
      }
    } ]
  }
}

```

GO TO A CONSOLE WHERE THE LOGSTASH DAEMON WAS STARTED DIRECTLY AND CANCEL IT.

Start Logstash service:
```
sudo service logstash start
```

