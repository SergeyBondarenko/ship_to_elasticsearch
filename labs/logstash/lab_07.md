# Lab 07. Advanced formatting with conditionals.

A. Configure Logstash.
_/etc/logstash/conf.d/trex_apache_logs_advanced_filtering.conf_
```
input {
    beats {
        port => "5044"
    }   
}
filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }   
    geoip {
      source => "clientip"
      target => "geoip"
      add_field => [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
      add_field => [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]
    }   

    if [geoip][continent_code] == "EU" and [geoip][country_name] == "Italy" {
      mutate {
        add_field => {
          custom_message => "Mare, montania, pizza, mandolino ..."
        }   
        remove_field => "city_name"
      }   
    }   

    if [custom_message] =~ /\bmandolino\b/ {
      mutate {
        add_field => {
          custom_action => "Let's have a party!"
        }   
      }
    }

    if [response] > "400" {
      mutate {
        add_field => {
          custom_reason => "Errors are possible. Check Apache config."
        }
      }
    }
}
output {
    stdout {
      codec => rubydebug
    }
    elasticsearch {
      hosts => [ "localhost:9200" ]
      index => "trex_apache_logs_advanced_filter-%{+yyyy.MM.dd}"
    }
}
```
Save file.

SHUTDOWN LOGSTASH SERVICE AND DAEMONS IF THEY ARE RUNNING. 

B. Start Logstash:
```
sudo bash /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/trex_apache_logs_advanced_filtering.conf --config.reload.automatic --path.settings /etc/logstash/
```

C. Check ES data:
```
curl -XGET localhost:9200/trex_apache_logs_advanced_filter-2017.03.13/_search?pretty
```
