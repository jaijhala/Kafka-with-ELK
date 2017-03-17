# Kafka-with-ELK

The objective of this repo is to learn how to integrate Kafka with the ELK Stack using FileBeat.
Below is the event flow we'll be implementing: Log files -> FileBeat -> Kafka -> Logstash -> Elasticsearch -> Kibana

Versions:
FileBeat 5.2.2
Kafka 0.10.0.1
Logstash 5.2.2
Elasticsearch 5.2.2
Kibana 5.2.2

All of these were installed on the same machine localhost. 

1). Filebeat -> We will be gathering the default logs - /var/log/*.log specified in filebeat.yml
Add following lines in the output section of filebeat.yml file :

output.kafka:
 enabled: true
 hosts: ["localhost:9092"]
 topic: test
compression: none

This will send events to Kafka port 9092 on topic test. 
Start Filebeat depending on the package/OS you have, more details -> https://www.elastic.co/guide/en/beats/filebeat/current/directory-layout.html
https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-command-line.html

2). Kafka -> Start the Kafka server and zookeeper using below commands:(https://kafka.apache.org/0100/documentation.html)

bin/kafka-server-start.sh config/server.properties

bin/zookeeper-server-start.sh config/zookeeper.properties

Create a topic called test ->
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

Filebeat will act as a producer. Start a console consumer as a test to see if you are able to consume those events: 
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning

At this point, you should be able to see lots of events in the console consumer. This test confirms that the setup is working fine so far. 

3). Logstash -> We will use Kafka input plugin in this case. 
Create logstash.conf file and add below lines :

input {
  kafka {
        topics => "test"
        }
}
output {
        stdout { codec => rubydebug }
}

Start Logstash specifying the above logstash.conf file. 
This will start consuming events from test topic. You should see lots of log files in the Logstash command line. Again the idea here is to make sure that Logstash is receiving and sending out those events as expected. 

4). Now start setting up Logstash to send to Elasticsearch

Add following lines in the logstash.conf output part -> 
elasticsearch {
                hosts => "localhost:9200"
                index => "filebeat"
                      }

This will send events to elasticsearch on the localhost and create the index filebeat. 

Make sure to start elasticsearch and Kibana at this point. 

5). Login to Kibana and create Index Pattern filebeat (uncheck the timestamp). 
Click on Discover to see the received events. Check the attached screenshot for a sample output. 


