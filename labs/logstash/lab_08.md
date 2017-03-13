# Lab 08. Advanced filtering with grok and regex patterns.

Define regexp patterns:
_configs/patterns/apache_
```
SRC_IPv4_ADDR \d+\.\d+\.\d+\.\d+
```

Parse message text with patterns:
```
filter {
  grok {
    patterns_dir => ["/home/trex/Development/ship_to_elasticsearch/configs/patterns"]
    match => {"message" => "%{SRC_IPv4_ADDR:src_ipv4_addr}"}
  }
}
```
