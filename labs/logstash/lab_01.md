# Lab 01. Install Lab Software.

[CentOS tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-elk-stack-on-centos-7)

## PostgreSQL

Install:
```
sudo yum install postgresql-server postgresql-contrib
```

Create DB cluster:
```
sudo postgresql-setup initdb
```

Allow password authentication:
_/var/lib/pgsql/data/pg_hba.conf_
```
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

Start and enable:
```
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

## Apache

Install and start:
```
sudo yum install httpd
sudo /usr/sbin/apachectl start
```

Configure to run automatically:
```
sudo /sbin/chkconfig httpd on
```

## Java 8

Download:
```
cd ~
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u73-b02/jdk-8u73-linux-x64.rpm"
```

Install:
```
sudo yum -y localinstall jdk-8u73-linux-x64.rpm
```

## Elasticsearch

Import the Elasticsearch public GPG key into rpm:
```
sudo rpm --import http://packages.elastic.co/GPG-KEY-elasticsearch
```

Create a new yum repository file for Elasticsearch. Note that this is a single command:
```
echo '[elasticsearch-2.x]
name=Elasticsearch repository for 2.x packages
baseurl=http://packages.elastic.co/elasticsearch/2.x/centos
gpgcheck=1
gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1
' | sudo tee /etc/yum.repos.d/elasticsearch.repo
```

Install:
```
sudo yum -y install elasticsearch
```

Configure:
_/etc/elasticsearch/elasticsearch.yml_
```
network.host: localhost
```

Start:
```
sudo systemctl start elasticsearch
```

Configure to start automatically:
```
sudo systemctl enable elasticsearch
```

## Logstash

Edit Yum repo file for Logstash:
_/etc/yum.repos.d/logstash.repo_
```
[logstash-5.0]
name=logstash repository for 5.0 packages
baseurl=http://packages.elasticsearch.org/logstash/5.0/centos
gpgcheck=1
gpgkey=http://packages.elasticsearch.org/GPG-KEY-elasticsearch
enabled=1
```

Install:
```
sudo yum -y install logstash
```

Start and configure to start automatically:
```
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch

```

## Filebeat

Create Yum repo file for Filebeat:
_/etc/yum.repos.d/elastic-beats.repo_
```
[beats]
name=Elastic Beats Repository
baseurl=https://packages.elastic.co/beats/yum/el/$basearch
enabled=1
gpgkey=https://packages.elastic.co/GPG-KEY-elasticsearch
gpgcheck=1
```

Install: 
```
sudo yum -y install filebeat
```

Configure:
_filebeat/filebeat.yml_
```
filebeat.prospectors:
- input_type: log
  paths:
    - /path/to/apache/file.log 
output.logstash:
  hosts: ["localhost:5044"]
```

Start and enable:
```
sudo systemctl start filebeat
sudo systemctl enable filebeat
```
