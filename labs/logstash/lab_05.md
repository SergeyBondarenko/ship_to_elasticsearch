# Lab 05. Get JSON data from file.

A. Configure Logstash.

_etc/logstash/conf.d/trex_shakespeare.conf_
```
input {
  file {
    type => "shakespeare"
    path => ["/home/centos/shakespeare.json"]
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}
output {
  stdout {
    codec => rubydebug
  }
}
```
Save the file. 

SHUTDOWN LOGSTASH SERVICE AND DAEMONS IF IT THEY ARE RUNNING. 
Run new Logstash daemon:
```
sudo bash /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/trex_shakespeare.conf --config.test_and_exit --path.settings /etc/logstash/
```
Cancel the daemon as soon as you see the import messages. Look at the imported documents, all plays dictionary content is a string value of the message key.


B. Add a filter to parse message and create new fields. Also add ES output.
```
filter {
  if [message] =~ /^{.*}$/ {
    json {
      source => message
    }   
  }
}
output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "trex_%{type}" 
  }   
}   
```
Save the file.

Check data in Elasticsearch:

```
curl -XGET localhost:9200/trex_shakespeare/_search?pretty -d '{"query": {"bool": {"must": [{"match": {"speaker": "hamlet"}}, {"match": {"text_entry": "lads"}}]}}}'
```

Now, you should see new fields created after message key was parsed by json plugin.