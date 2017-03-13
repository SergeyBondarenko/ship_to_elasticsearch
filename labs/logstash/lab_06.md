# Lab 06. Get Database data.
  
## 1. Configure Database.
A. Access Database: 
```
sudo -u postgres psql postgres
```
B. Setting up Database:
```
create user logstash;
alter user logstash password 'logstashpass';
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO logstash;
```
C. Create table:
```
create table contacts (
    uid serial,
    timestamp timestamp NOT NULL,
    email VARCHAR(80) NULL,
    first_name VARCHAR(80) NULL,
    last_name VARCHAR(80) NULL
);
```

## 2. Insert Data into Database.
```
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp,  'jim@example.com', 'Jim', 'Smith');
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp, null, 'John', 'Smith');
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp, 'carol@example.com', 'Carol', 'Smith');
INSERT INTO contacts(timestamp, email, first_name, last_name) VALUES(current_timestamp, 'sam@example.com', 'Sam', null);
```

## 3. Configure Logstash.

_configs/logstash/trex_postgresql.conf_
```
input {
  jdbc {
    # Postgres jdbc connection string to our database, mydb
    jdbc_connection_string => "jdbc:postgresql://localhost:5432/postgres"
    # The user we wish to execute our statement as
    jdbc_user => "logstash"
    jdbc_password => "logstashpass"
    # The path to our downloaded jdbc driver
    jdbc_driver_library => "/home/centos/postgresql-42.0.0.jar"
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
    index => "trex_database.%{+yyyy.MM.dd}"
  }
}
```

SHUTDOWN LOGSTASH SERVICE AND DAEMONS IF THEY ARE RUNNING. 

Run Logstash: 
```
sudo bash /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/trex_postgresql.conf --config.reload.automatic --path.settings /etc/logstash/
```

Check data:
```
curl -XGET localhost:9200/trex_database.2017.03.13/_search?pretty
curl -XGET localhost:9200/trex_database.2017.03.13/_count?pretty
```

Check Logstash SQL last query run time: `sudo cat /root/.logstash_jdbc_last_run`

