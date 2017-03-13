# Ship to Elasticsearch

Exploring different ways to ship data for Elasticsearch.

## Data.

[William Shakespeare, collection of plays](https://www.elastic.co/guide/en/kibana/3.0/snippets/shakespeare.json)

## Software.
* [Apache WEB Server](https://www.cyberciti.biz/faq/linux-install-and-start-apache-httpd/)
* [Filebeat](https://www.elastic.co/downloads/beats/filebeat)
* [Logstash](https://www.elastic.co/downloads/logstash)
* [Elasticsearch](https://www.elastic.co/downloads/elasticsearch)
* [PostgreSQL](https://www.cyberciti.biz/faq/linux-installing-postgresql-database-server/)
* [PostgreSQL JDBC driver](https://jdbc.postgresql.org/)

## Configs.
```
git clone https://github.com/SergeyBondarenko/ship_to_elasticsearch.git
```

## Eastic Stack.
![Elastic Stack](https://github.com/SergeyBondarenko/ship_to_elasticsearch/blob/master/assets/img/elastic_stack.png)

## Logstash.

### How It Works.  

![Basic Logstash Pipeline](https://github.com/SergeyBondarenko/ship_to_elasticsearch/blob/master/assets/img/basic_logstash_pipeline.png)

Inputs:
* file
* syslog
* redis
* beats: filebeat, metricbeat, packetbeat, winlogbeat, heartbeat
* database (jdbc)
* ...

Filters:
* grok
* mutate
* drop
* clone
* geoip

Outputs:
* elasticsearch
* file
* graphite
* statsd
* ...

Codecs:
* json
* sflow
* netflow
* nmap
* ...

### Stashing Event

```
bin/logstash -e 'input { stdin { } } output { stdout {} }'
```

After starting Logstash, wait until you see "Pipeline main started" and then enter hello world at the command prompt:

```
hello world
```

Output: `2013-11-21T01:22:14.405+0000 0.0.0.0 hello world`

### Parse Apache Logs.

#### 1. Configure Filebeat.
Before you create the Logstash pipeline, you’ll configure Filebeat to send log lines to Logstash.

_filebeat/filebeat.yml_
```
filebeat.prospectors:
- input_type: log
  paths:
    - /path/to/apache/file.log 
output.logstash:
  hosts: ["localhost:5044"]
```

Run Filebeat: `sudo ./filebeat -e -c filebeat.yml -d "publish"`

#### 2. Configure Logstash.
_configs/logstash/beats.conf_
```
input {
    beats {
        port => "5043"
    }
}
output {
    stdout { codec => rubydebug }
}
```

Test config: `bin/logstash -f beats.conf --config.test_and_exit` 

Run Logstash: `bin/logstash -f beats.conf --config.reload.automatic` 

Parse Apache logs with grok plugin.
```
...
filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
}
...
```

Index Data into Elasticseaerch.

```
...
output {
    elasticsearch {
        hosts => [ "localhost:9200" ]
    }
}
```

### Get JSON Data From a File
```
input {
  file {
    type => "shakespeare"
    path => ["/home/trex/Development/Shipping_Data_To_ES/shakespeare.json"]
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}
output {
  stdout {
    codec => rubydebug
  }
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "%{type}" 
  }   
}
```

Parse JSON message:
```
filter {
  if [message] =~ /^{.*}$/ {
    json {
      source => message
    }   
  }
}
```

Check data in Elasticsearch:
```
{"query": {"bool": {"must": [{"match": {"speaker": "hamlet"}}, {"match": {"text_entry": "lads"}}]}}}
```

### Get Data From Database.

#### 1. Configure Database.
Access Database: 
```
sudo -u postgres psql postgres
```
Setting up Database:
```
create user logstash;
alter user logstash password 'logstashpass';
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO logstash;
```
Create table:
```
create table contacts (
    uid serial,
    timestamp timestamp NOT NULL,
    email VARCHAR(80) NULL,
    first_name VARCHAR(80) NULL,
    last_name VARCHAR(80) NULL
);
```

#### 2. Insert Data into Database.
```
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp,  'jim@example.com', 'Jim', 'Smith');
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp, null, 'John', 'Smith');
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp, 'carol@example.com', 'Carol', 'Smith');
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp, 'sam@example.com', 'Sam', null);
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp, 'sbondarenko@example.com', 'Sergii', 'Bondarenko');
```

#### 3. Configure Logstash.

_configs/logstash/postgresql.conf_
```
input {
  jdbc {
    # Postgres jdbc connection string to our database, mydb
    jdbc_connection_string => "jdbc:postgresql://localhost:5432/postgres"
    # The user we wish to execute our statement as
    jdbc_user => "logstash"
    jdbc_password => "logstashpass"
    # The path to our downloaded jdbc driver
    jdbc_driver_library => "/home/trex/Development/ship_to_elasticsearch/software/postgresql-42.0.0.jar"
    # The name of the driver class for Postgresql
    jdbc_driver_class => "org.postgresql.Driver"
    # Time zone
    jdbc_default_timezone => "Europe/Rome"
    # our query
    statement => "SELECT * FROM contacts WHERE timestamp > :sql_last_value"
    # disable sql_las_value
    #clean_run => true
    # every 1 min
    schedule => "*/1 * * * *"
  }
}
output {
  stdout { codec => json_lines }
  elasticsearch {
    hosts => [ "localhost:9200" ]
    index => "database.%{+yyyy.MM.dd}"
  }
}
```

Run Logstash: 
```
./bin/logstash -f configs/logstash/postgresql.conf --config.reload.automatic
```

Check data:
```
curl -XGET localhost:9200/database.2017.03.07.15/_search?pretty
curl -XGET localhost:9200/database.2017.03.07.15/_count?pretty
```

Check Logstash SQL last query run time: `cat ~/.logstash_jdbc_last_run`

### Setting up and Running Logstash
Directory Layout
* Bin
* Settings
* Logs
* Plugins

Configuration files
* Logstash.yml
* Jvm.options
* Startup.options

#### Running Logstash as a Service
* Systemd: `sudo systemctl start logstash.service`
* Upstart: `sudo initctl start logstash`
* SysV: `sudo /etc/init.d/logstash start`


### Deploying and Scaling Logstash
...

### Performance Tuning.
...

### Environment Variables.
```
input {
  beats {
    type => "filebeat"
    port => "${BEATS_PORT:5044}"
  }
}
```

### Accessing Fields Data.
#### Conditionals.
```
if "Latitude-E5510" in [host] {
  mutate {
    add_field => {
      computer_type => "super_computer"
    }
  } 
}
if [computer_type] =~ /super.+/ {
  mutate {
    add_field => {
      alarm => "green"
    }
  } 
}
if [response] == "200" {
  mutate {
    add_field => {
      request_type => "GET"
    }
    remove_field => "verb"
  } 
}
if [beat][hostname] == "Latitude-E5510" and [clientip] == "127.0.0.1" and !("red" == [alarm]) {
  mutate {
    add_field => {
      custom_message => "TRUE LOCALHOST ORIGIN" 
    }
  }
}
```

#### Grok plugin.

Define regexp patterns:
_configs/patterns/apache_
```
DEST_IPv4_ADDR \d+\.\d+\.\d+\.\d+
DEST_PORT \d+
SRC_IPv4_ADDR \d+\.\d+\.\d+\.\d+
```

Parse message text with patterns:
```
filter {
  grok {
    patterns_dir => ["/home/trex/Development/ship_to_elasticsearch/configs/patterns"]
    match => {"message" => "%{DEST_IPv4_ADDR:dest_ipv4_addr}:%{DEST_PORT:dest_port} %{SRC_IPv4_ADDR:src_ipv4_addr}"}
  }
}
```

## Authors

**Sergii Bondarenko**

