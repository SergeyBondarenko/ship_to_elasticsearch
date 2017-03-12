<h1> Ship To Elasticsearch</h1>

Exploring different ways to ship data for Elasticsearch.

<h2>Data:</h2>
[William Shakespeare, collection of plays](https://www.elastic.co/guide/en/kibana/3.0/snippets/shakespeare.json)

<h2>Software:</h2>
<ul>
<li>
[Apache WEB Server](https://www.cyberciti.biz/faq/linux-install-and-start-apache-httpd/)
</li>
<li>
[Filebeat](https://www.elastic.co/downloads/beats/filebeat)
</li>
<li>
[Logstash](https://www.elastic.co/downloads/logstash)
</li>
<li>
[Elasticsearch](https://www.elastic.co/downloads/elasticsearch)
</li>
<li>
[PostgreSQL](https://www.cyberciti.biz/faq/linux-installing-postgresql-database-server/)
</li>
<li>
[PostgreSQL JDBC driver](https://jdbc.postgresql.org/)
</li>
</ul>

<h2>Configs:</h2>
<code>
git clone https://github.com/SergeyBondarenko/ship_to_elasticsearch.git
</code>


<h2>Logstash.</h2>

<h3>Getting Started with Logstash.</h3>

<h4>Stashing Your First Event.</h4>

To test your Logstash installation, run the most basic Logstash pipeline:

<code>
cd logstash-5.2.1
bin/logstash -e 'input { stdin { } } output { stdout {} }'
</code>

After starting Logstash, wait until you see "Pipeline main started" and then enter hello world at the command prompt:

<code>
hello world
</code>

Output:
<code>
2013-11-21T01:22:14.405+0000 0.0.0.0 hello world
</code>

<h4>Parsing Logs With Logstash.</h4>

<h3>1. Configuring Filebeat.</h3>
Before you create the Logstash pipeline, you’ll configure Filebeat to send log lines to Logstash. To install Filebeat on your data source machine, download the appropriate package from the Filebeat product page.

<p><b>filebeat/filebeat.yml</b></p>
<pre>
<code>
filebeat.prospectors:
- input_type: log
  paths:
    - /path/to/apache/file.log 
output.logstash:
  hosts: ["localhost:5044"]
</code>
</pre>

Run:
sudo ./filebeat -e -c filebeat.yml -d "publish"

Configuring Logstash for Filebeat.
logstash/config/first-pipeline.conf
input {
    beats {
        port => "5043"
    }
}
output {
    stdout { codec => rubydebug }
}

Test config:
bin/logstash -f first-pipeline.conf --config.test_and_exit

Run Logstash:
bin/logstash -f first-pipeline.conf --config.reload.automatic

Parsing Web Logs with the Grok Filter Plugin
logstash/config/first-pipeline.conf
input {
    beats {
        port => "5044"
    }
}
filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
}
output {
    stdout { codec => rubydebug }
}

Then delete the Filebeat registry file. For example, run:
sudo rm data/registry

Next, restart Filebeat with the following command:
sudo ./filebeat -e -c filebeat.yml -d "publish"

Indexing Your Data into Elasticsearch
Download Elasticsearch.

logstash/config/first-pipeline.conf
output {
    elasticsearch {
        hosts => [ "localhost:9200" ]
    }
}

Then delete the Filebeat registry file. For example, run:
sudo rm data/registry

Next, restart Filebeat with the following command:
sudo ./filebeat -e -c filebeat.yml -d "publish"

Check data in Elasticsearch:
curl -XGET 'localhost:9200/logstash-$DATE/_search?pretty&q=response=200'
How Logstash Works
Inputs:
File
Syslog
Redis
Beats

Filters:
Grok
Mutate
Drop
Clone
Geoip

Outputs:
Elasticsearch
File
…

Setting up and Running Logstash
Directory Layout

Bin
Settings
Logs
Plugins

Configuration files

Logstash.yml
Jvm.options
Startup.options


Running Logstash as a Service

Systemd: sudo systemctl start logstash.service
Upstart: sudo initctl start logstash
SysV: sudo /etc/init.d/logstash start
Insert JSON Data From a File 
Basic
input {
  file {
    path => ["/home/trex/Development/Shipping_Data_To_ES/shakespeare.json"]
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}
output {
  stdout {
    codec => json_lines
  }
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "shakespeare" 
  }   
}

Advanced
input {
  file {
    type => "shakespeare"
    path => ["/home/trex/Development/Shipping_Data_To_ES/shakespeare.json"]
    start_position => "beginning"
    sincedb_path => "/home/trex/.logstash_sincedb_path"
  }
}
filter {
  # get JSON keys/values from message field and put them to _source
  if [message] =~ /^{.*}$/ {
    json {
      source => message
    }   
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

Check data:
{"query": {"bool": {"must": [{"match": {"speaker": "hamlet"}}, {"match": {"text_entry": "lads"}}]}}}
Insert Data From Database
https://www.elastic.co/blog/logstash-jdbc-input-plugin

Access Database
sudo -u postgres psql postgres
Setting up Database
create user logstash;
alter user logstash password 'logstashpass';
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO logstash;
create table contacts (
    uid serial,
    timestamp timestamp NOT NULL,
    email VARCHAR(80) NULL,
    first_name VARCHAR(80) NULL,
    last_name VARCHAR(80) NULL
);
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp,  'jim@example.com', 'Jim', 'Smith');
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp, null, 'John', 'Smith');
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp, 'carol@example.com', 'Carol', 'Smith');
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp, 'sam@example.com', 'Sam', null);

INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp, 'sbondarenko@example.com', 'Sergii', 'Bondarenko');

Check DB data 
select * from "contacts";
 
Index DB data into ES:
./software/logstash-5.2.1/bin/logstash -f logstash/config/postgresql.conf --config.reload.automatic

Check data:
curl -XGET localhost:9200/database.2017.03.07.15/_search?pretty
curl -XGET localhost:9200/database.2017.03.07.15/_count?pretty

Check Logstash sql last run time:
cat ~/.logstash_jdbc_last_run


Accessing Event Data and Fields
Environment variables
Conditionals

Plugins
grok

