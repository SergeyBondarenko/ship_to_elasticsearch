# Lab 04. Parse logs.

## 1. Configure Filebeat.
Before you create the Logstash pipeline, you’ll configure Filebeat to send log lines to Logstash.

_filebeat/filebeat.yml_
```
filebeat.prospectors:
- input_type: log
  paths:
    - /var/log/httpd/access_log 
output:
  logstash:
    hosts: ["localhost:5044"]
```
Save the file.

Restart Filebeat: `sudo service filebeat restart`

## 2. Start Logstash daemon manualy.
SHUTDOWN LOGSTASH SERVICE AND DAEMONS IF THEY ARE RUNNING. 
```
sudo bash /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/trex_apache_logs.conf --config.test_and_exit --path.settings /etc/logstash/
```

## 3. Configure Logstash.

A. Add basic configuration. Add this to the config and save the file.
_configs/logstash/trex_apache_logs.conf_
```
input {
    beats {
        port => "5044"
    }
}
output {
    stdout { codec => rubydebug }
}
```
Save the file.

Send HTTP GET request to the server 80 port and look at the Logstash daemon console output.

B. Parse Apache logs with grok plugin. Add this to the config and save the file.
```
...
filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
}
...
```
Save the file.

Look at the Logstash console output the message string should be parsed and new fields should be created.

C. Add geographic information to filter. Add this to the config and save the file.
```
geoip {
    source => "clientip"
    target => "geoip"
    add_field => [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
    add_field => [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]
}
```
Save the file.

Look at the Logstash console and see the geo information.

D. Index Data into Elasticsearch. Add this to the config and save the file.

```
...
output {
    elasticsearch {
        hosts => [ "localhost:9200" ]
        index => "trex_apache_logs-%{+yyyy.MM.dd}"
    }
}
```
Save the file.

Check ES data: `curl -XGET localhost:9200/trex_apache_logs-2017.03.13/_search?pretty`
