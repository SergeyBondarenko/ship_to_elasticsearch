<h1> Ship To Elasticsearch</h1>

Exploring different ways to ship data for Elasticsearch.

<h2>Data:</h2>
[William Shakespeare, collection of plays](https://www.elastic.co/guide/en/kibana/3.0/snippets/shakespeare.json)

<h2>Software:</h2>
[Apache WEB Server](https://www.cyberciti.biz/faq/linux-install-and-start-apache-httpd/)

[Filebeat](https://www.elastic.co/downloads/beats/filebeat)

[Logstash](https://www.elastic.co/downloads/logstash)

[Elasticsearch](https://www.elastic.co/downloads/elasticsearch)

[PostgreSQL](https://www.cyberciti.biz/faq/linux-installing-postgresql-database-server/)

[PostgreSQL JDBC driver](https://jdbc.postgresql.org/)

<h2>Configs:</h2>
<code>
git clone https://github.com/SergeyBondarenko/ship_to_elasticsearch.git
</code>

<h2>Logstash 5.</h2>

<h3>Getting Started with Logstash.</h3>

<h4>Stashing Your First Event.</h4>

To test your Logstash installation, run the most basic Logstash pipeline:

<code>
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
Before you create the Logstash pipeline, you’ll configure Filebeat to send log lines to Logstash.

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

Run Filebeat:

<code>
sudo ./filebeat -e -c filebeat.yml -d "publish"
</code>

<h3>2. Configuring Logstash for Filebeat.</h3>

<p><b>logstash/config/beats.conf</b></p>
<pre>
<code>
input {
    beats {
        port => "5043"
    }
}
output {
    stdout { codec => rubydebug }
}
</code>
</pre>

Test config:
<code>
bin/logstash -f first-pipeline.conf --config.test_and_exit
</code>

Run Logstash:
<code>
bin/logstash -f first-pipeline.conf --config.reload.automatic
</code>

Parsing Web Logs with the Grok Filter Plugin

<p><b>logstash/config/beats.conf</b></p>
<pre>
<code>
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
</code>
</pre>

<h3>3. Indexing Your Data into Elasticsearch.</h3>

<p><b>logstash/config/beats.conf</b></p>
<pre>
<code>
output {
    elasticsearch {
        hosts => [ "localhost:9200" ]
    }
}
</code>
</pre>

Check data in Elasticsearch:
<code>
curl -XGET 'localhost:9200/logstash-$DATE/_search?pretty&q=response=200'
</code>


<h4>How Logstash Works.</h4>

Inputs:
<ul>
<li>File
</li>
<li>Syslog
</li>
<li>Redis
</li>
<li>Beats
</li>
</ul>

Filters:
<ul>
<li>Grok
</li>
<li>Mutate
</li>
<li>Drop
</li>
<li>Clone
</li>
<li>Geoip
</li>
</ul>

Outputs:
<ul>
<li>Elasticsearch
</li>
<li>File
</li>
<li>…
</li>
</ul>

<h4>Setting up and Running Logstash.</h4>

Directory Layout
<ul>
<li>Bin
</li>
<li>Settings
</li>
<li>Logs
</li>
<li>Plugins
</li>
</ul>

Configuration files
<ul>
<li>Logstash.yml
</li>
<li>Jvm.options
</li>
<li>Startup.options
</li>
</ul>

<h4>Running Logstash as a Service.</h4>

Systemd: 
<code>
sudo systemctl start logstash.service
</code>
Upstart: 
<code>
sudo initctl start logstash
</code>
SysV: 
<code>
sudo /etc/init.d/logstash start
</code>


<h4>Insert JSON Data From a File.</h4>

Basic

<p><b>config/logstash/json_file.conf</b></p>
<pre>
<code>
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
</code>
</pre>

Parsing JSON message.

<pre>
<code>
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
</code>
</pre>

Checking data:
<code>
{"query": {"bool": {"must": [{"match": {"speaker": "hamlet"}}, {"match": {"text_entry": "lads"}}]}}}
</code>

<h4>Insert Data From Database.</h4>

<h3>1. Configure Database.</h3>
Access Database
<code>
sudo -u postgres psql postgres
</code>
Setting up Database
<pre>
<code>
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
</code>
</pre>

<h3>2. Insert Data into Database.</h3>
<pre>
<code>
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp,  'jim@example.com', 'Jim', 'Smith');
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp, null, 'John', 'Smith');
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp, 'carol@example.com', 'Carol', 'Smith');
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp, 'sam@example.com', 'Sam', null);
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp, 'sbondarenko@example.com', 'Sergii', 'Bondarenko');
</code>
</pre>

Check DB data 
<code>
select * from "contacts";
</code>
 
<h3>3. Configure Logstash.</h3>

<p><b>configs/logstash/postgresql.conf</b></p>
<pre>
<code>
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
    index => "database.%{+yyyy.MM.dd.HH}"
  }
}
</code>
</pre>

<h3>4. Index DB data into ES.</h3>
<code>
./bin/logstash -f logstash/config/postgresql.conf --config.reload.automatic
</code>

Check data:
<pre>
<code>
curl -XGET localhost:9200/database.2017.03.07.15/_search?pretty
curl -XGET localhost:9200/database.2017.03.07.15/_count?pretty
</code>
</pre>

Check Logstash sql last run time:
<code>
cat ~/.logstash_jdbc_last_run
</code>

<h4>Environment Variables.</h4>
<pre>
<code>
input {
  beats {
    type => "filebeat"
    port => "${BEATS_PORT:5044}"
  }
}
</code>
</pre>

<h4>Accessing Event Data and Fields.</h4>

<h3>Conditionals.</h3>

<pre>
<code>
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
</code>
</pre>

<h3>Grok plugin.</h3>

Define patterns in a file inside <b>configs/patterns</b>.
<pre>
<code>
DEST_IPv4_ADDR \d+\.\d+\.\d+\.\d+
DEST_PORT \d+
SRC_IPv4_ADDR \d+\.\d+\.\d+\.\d+
</code>
</pre>

Parse message with patterns.
<pre>
<code>
filter {
  grok {
    patterns_dir => ["/home/trex/Development/ship_to_elasticsearch/configs/patterns"]
    match => {"message" => "%{DEST_IPv4_ADDR:dest_ipv4_addr}:%{DEST_PORT:dest_port} %{SRC_IPv4_ADDR:src_ipv4_addr}"}
  }
}
</code>
</pre>
