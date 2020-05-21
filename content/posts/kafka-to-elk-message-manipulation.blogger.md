---
title: "Kafka to Elk Message Manipulation"
date: 2020-05-21T05:58:11+01:00
draft: true
tags: ["docker", "kafka", "elasticSearch", "kibana", "logstash", "ELK", "ruby"]
---

<!--- Below style are also defined in static/css/my.css file.
They are repeatedly defined here so that pandoc can generate
the final HTML with all necessary css styles.
Note: draft: true above. This prevents publishing it to GitHUB.
--->
<style>
/* To highlight text in Green in pre tag */
.hl {color: #008A00;}
/* To highlight text in Bold Green in pre tag */
.hlb {color: #008A00; font-weight: bold;}
/* To highlight text in Bold Red in pre tag */
.hlbr {color:#e90001; font-weight: bold;}
/* <code> tag does not work in blogger. Use following class with span tag */
.code {
    color:#7e168d; 
    background: #f0f0f0; 
    padding: 0.1em 0.4em;
    font-family: SFMono-Regular, Consolas, "Liberation Mono", Menlo, Courier, monospace;
}
</style>

## Introduction
We will be creating a message flow starting from Kafka and ending in Kibana. Flow will be like below:

`console app (to send json message) -> kafka -> logstash -> elasticSearch -> kibana`

We will be using ruby filter to manipulate the message as well and docker to setup the environment.

## Setup

```bash
mkdir kafka-elk
cd kafka-elk
wget http://apache.mirror.anlx.net/kafka/2.5.0/kafka_2.13-2.5.0.tgz
tar xvf kafka_2.13-2.5.0.tgz
```
Note: We need kafka binaries to get the script that can send json message via console to kafka.

### Populate `docker-compose.yml` file with following contents

```yml
version: '2'

services:
  zookeeper:
    image: 'bitnami/zookeeper:3'
    container_name: zookeeper
    ports:
      - '2181:2181'
    volumes:
      - 'zookeeper_data:/bitnami'
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
    networks:
      - elastic

  kafka:
    image: 'bitnami/kafka:2.5.0'
    container_name: kafka
    ports:
      - '9092:9092'
      - '29092:29092'
    volumes:
      - 'kafka_data:/bitnami'
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,PLAINTEXT_HOST://:29092
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
    depends_on:
      - zookeeper
    networks:
      - elastic

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.6.2
    container_name: elasticsearch
    environment:
      - node.name=elasticsearch
      - cluster.name=es-docker-cluster
      - cluster.initial_master_nodes=elasticsearch
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - elastic_data:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic

  kibana:
    image: docker.elastic.co/kibana/kibana:7.6.2
    container_name: kibana
    ports:
      - 5601:5601
    environment:
      ELASTICSEARCH_URL: http://elasticsearch:9200
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200
    networks:
      - elastic

  logstash:
    image: docker.elastic.co/logstash/logstash:7.7.0
    container_name: logstash
    ports:
      - 5000:5000
    volumes:
    - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    - ./logstash.yml:/usr/share/logstash/config/logstash.yml
    - ./manipulate_msg.rb:/etc/logstash/manipulate_msg.rb
    networks:
      - elastic

networks:
  elastic:
    driver: bridge

volumes:
  zookeeper_data:
    driver: local
  kafka_data:
    driver: local
  elastic_data:
    driver: local
```

### create `logstash.conf` file
```bash
cat <<EOF > logstash.conf
input {
  kafka {
    bootstrap_servers => "kafka:9092"
    client_id => "transform-text"
    group_id => "transform-text"
    consumer_threads => 3
    topics => ["transform-text"]

    # Following multiline json codec may not work on all the
    # possible multiline json records.
    # codec => multiline {
    #  pattern => "^\{"
    #  negate => true
    #  what => previous
    # }

    # use json record with no newline in between.
    codec => json
    tags => ["transformed-text", "kafka_source"]
    type => "kafka-test-messages"
  }

  # to test the logstah via telnet
  # e.g. cat some.json | nc localhost 5000
  tcp {
    port => 5000
    type => syslog
    codec => multiline {
      pattern => "^\{$"
      negate => true
      what => previous
    }
  }

  # to test the logstah via telnet
  # e.g. cat some.json | nc localhost 5000
  udp {
    port => 5000
    type => syslog
    codec => multiline {
      pattern => "^\{$"
      negate => true
      what => previous
    }
  }

}

filter {
  ruby {
    path => "/etc/logstash/manipulate_msg.rb"
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "logstash-%{+YYYY.MM.dd}"
  }
  stdout { codec => rubydebug }
}
EOF
```

### create `logstash.yml` with following contents
```yml
http.host: "0.0.0.0"
config.support_escapes: true
```

### create `manipulate_msg.rb` with following contents.
```ruby
def filter(event)
  # get the message line sent by kafka or any other source like syslog
  message = event.get("message")
  event.set("newField", "newValue")
  return [event]
end
```
**Note:** We are manipulating the message coming from kafka by adding an extra field/value pair to the message in logstash ruby filter which will be visible in kibana.

**Note** Kafka is not pushing messages to logstash. Its the logstash that pulling messages from kafka (acting as kafka consumer). 

### Create a `sample.json` file with following contents
```json
{
  "menu": {
    "id": "file with space",
    "value": "File",
    "popup": {
      "menuitem": [
        {"value": "New", "onclick": "CreateNewDoc()"},
        {"value": "Open", "onclick": "OpenDoc()"},
        {"value": "Close", "onclick": "CloseDoc()"}
      ]
    }
  }
}
```
**Note:** We need to make above json input to be in one-line, as logstash cannot ingest multiline json record. You can use multiline codec in logstash.conf input plugin (to ingest multiline json) but then kibana will show that record as a single string and json record's field/keys will not be shown as individual fields in kibana.

Multiline codec will make json record just a series of characters (a long string) and record's keys/fields will not be recoginzed in kibana as separate fields. You might need to make changes to ruby filter so that kibana can show record's key/fields as individual searchable fields.

To keep things simple, we will be converting our json in a flat structure and use codec json in logstash.conf's kafka input plugin.

### flatten the json record.
```bash
cat sample.json |  perl -wp -e 's/\n+//g' > flat_sample.json
```

## Start the whole setup
```bash
docker-compose up -d
```
**Note**: You can bring down whole setup by running this command `docker-compose down -v`

## Setup kibana
Wait for a minute for above setup to come up fully, and open Kibana URL: http://localhost:5601/ . You need to create an index pattern with logstash-* as index patten (inside management). But before you can create index-pattern you need to send some data to elastic search. That can be sent via running below mentioned kafka-console-producer.sh command. Once you have sent the json , you should be able to create index pattern. Now click on "discover" to view the sent data. Try sending more data.

## send json data via kafka
```bash
kafka_2.13-2.5.0/bin/kafka-console-producer.sh --topic "transform-text" --bootstrap-server localhost:29092 < flat_sample.json
```
**Note**: Kafka input plugin is using json codec

**Note:** You should see json data visible in Kibana.

**Note** You can see data being ingested by logstash by viewing logstash logs `docker logs -f logstash`

## send json data via syslog port 5000
```bash
 cat flat_sample.json | nc localhost 5000
```
**Note:** tcp input plugin is using port 5000 which is can be used by syslog as well. 

**Note:** above command is sending data straight to logstash (skipping kafka)

**Note:** tcp input plugin is using multiline codec.

Please note the difference in reprsentation of two records sent above in kibana in order to understand the differnce between multiline and json codec. 




