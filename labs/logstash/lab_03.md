# Lab 03. Discover Logstash status.

## Node information:
```
$ curl -XGET localhost:9600/_node?pretty
{
  "host" : "ip-10-0-0-212.eu-west-1.compute.internal",
  "version" : "5.0.0-alpha5",
  "http_address" : "127.0.0.1:9600",
  "pipeline" : {
    "workers" : 4,
    "batch_size" : 125,
    "batch_delay" : 5
  },
  "os" : {
    "name" : "Linux",
    "arch" : "amd64",
    "version" : "3.10.0-327.10.1.el7.x86_64",
    "available_processors" : 4
  },
  "jvm" : {
    "pid" : 4408,
    "version" : "1.8.0_121",
    "vm_name" : "Java HotSpot(TM) 64-Bit Server VM",
    "vm_version" : "1.8.0_121",
    "vm_vendor" : "Oracle Corporation",
    "start_time_in_millis" : 1489412192357,
    "mem" : {
      "heap_init_in_bytes" : 268435456,
      "heap_max_in_bytes" : 1038876672,
      "non_heap_init_in_bytes" : 2555904,
      "non_heap_max_in_bytes" : 0
    }
  }
```

## Deep statistics:
```
$ curl -XGET localhost:9600/_node/stats?pretty
{
  "host" : "ip-10-0-0-212.eu-west-1.compute.internal",
  "version" : "5.0.0-alpha5",
  "http_address" : "127.0.0.1:9600",
  "jvm" : {
    "threads" : {
      "count" : 25,
      "peak_count" : 26
    },
    "mem" : {
      "heap_used_in_bytes" : 410042112,
      "heap_used_percent" : 19,
      "heap_committed_in_bytes" : 519045120,
      "heap_max_in_bytes" : 2077753344,
      "non_heap_used_in_bytes" : 171433648,
      "non_heap_committed_in_bytes" : 180936704,
      "pools" : {
        "survivor" : {
          "peak_used_in_bytes" : 8912896,
          "used_in_bytes" : 10812592,
          "peak_max_in_bytes" : 34865152,
          "max_in_bytes" : 69730304,
          "committed_in_bytes" : 17825792
        },
        "old" : {
          "peak_used_in_bytes" : 148135608,
          "used_in_bytes" : 280167016,
          "peak_max_in_bytes" : 724828160,
          "max_in_bytes" : 1449656320,
          "committed_in_bytes" : 357957632
        },
        "young" : {
          "peak_used_in_bytes" : 71630848,
          "used_in_bytes" : 119062504,
          "peak_max_in_bytes" : 279183360,
          "max_in_bytes" : 558366720,
          "committed_in_bytes" : 143261696
        }
      }
    }
  },
  "process" : {
    "open_file_descriptors" : 56,
    "peak_open_file_descriptors" : 56,
    "max_file_descriptors" : 4096,
    "mem" : {
      "total_virtual_in_bytes" : 4832329728
    },
    "cpu" : {
      "total_in_millis" : 41550000000,
      "percent" : 0
    }
  },
  "mem" : {
    "heap_used_in_bytes" : 410042112,
    "heap_used_percent" : 19,
    "heap_committed_in_bytes" : 519045120,
    "heap_max_in_bytes" : 2077753344,
    "non_heap_used_in_bytes" : 171433648,
    "non_heap_committed_in_bytes" : 180936704,
    "pools" : {
      "survivor" : {
        "peak_used_in_bytes" : 8912896,
        "used_in_bytes" : 10812592,
        "peak_max_in_bytes" : 34865152,
        "max_in_bytes" : 69730304,
        "committed_in_bytes" : 17825792
      },
      "old" : {
        "peak_used_in_bytes" : 148135608,
        "used_in_bytes" : 280167016,
        "peak_max_in_bytes" : 724828160,
        "max_in_bytes" : 1449656320,
        "committed_in_bytes" : 357957632
      },
      "young" : {
        "peak_used_in_bytes" : 71630848,
        "used_in_bytes" : 119062504,
        "peak_max_in_bytes" : 279183360,
        "max_in_bytes" : 558366720,
        "committed_in_bytes" : 143261696
      }
    }
  },
  "pipeline" : {
    "events" : {
      "in" : 1,
      "filtered" : 1,
      "out" : 1
    },
    "plugins" : {
      "inputs" : [ ],
      "filters" : [ {
        "id" : "grok_9830d507-f3ba-43c9-918c-3fd6685d7ba7",
        "events" : {
          "duration_in_millis" : 1,
          "in" : 1,
          "out" : 1
        },
        "matches" : 1,
        "patterns_per_field" : {
          "message" : 1
        },
        "name" : "grok"
      } ],
      "outputs" : [ {
        "id" : "elasticsearch_bcaddc7b-5ecc-40e0-a1f0-3e653b67e081",
        "events" : {
          "duration_in_millis" : 44,
          "in" : 1,
          "out" : 1
        },
        "name" : "elasticsearch"
      }, {
        "id" : "stdout_ba09d094-4fe5-4a73-bb33-37e13d417c00",
        "events" : {
          "duration_in_millis" : 3,
          "in" : 1,
          "out" : 1
        },
        "name" : "stdout"  
      } ]
    }
  }
} 
