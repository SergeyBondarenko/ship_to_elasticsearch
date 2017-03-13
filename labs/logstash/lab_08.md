# Lab 08. Advanced filtering with grok and regex patterns.

A. Define regexp patterns:
_configs/patterns/apache_
```
SRC_IPv4_ADDR \d+\.\d+\.\d+\.\d+
```

B. Parse messsage with custom patterns:
_/etc/logstash/conf.d/trex_apache_logs_advanced_filtering.conf_
```
filter {
  grok {
    patterns_dir => ["/home/trex/Development/ship_to_elasticsearch/configs/patterns"]
    match => {"message" => "%{SRC_IPv4_ADDR:src_ipv4_addr}"}
  }
}
```
