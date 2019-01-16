# Avro Passthrough Stream Processor with check for "retired" messages

This example shows the solution for checking for keyed messages which are older (retired) than previous messages. In order to function correctly, messages have to be keyed by an identifying property.
Both the key and the value part of the message currently have to be formatted in Avro and therefore the confluent schema registry has to be available.

## Using Docker environment

If you don't have a Kafka environment with all the necessary components (Kafka, Zookeeper and Schema Registry) running, you can start one up using Docker.

Before you can start the containers using docker-compose, two environment variables have to be setup. They have to be set to the IP address of the docker host and to the public IP address of the docker host (on a local machine they can be set to the same value.

```
export DOCKER_HOST_IP = nnn.nnn.nnn.nnn
export PUBLIC_IP = nnn.nnn.nnn.nnnn
```

Now navigate to the docker folder and start the environment using

```
cd ./docker
docker-compose up -d
```

After a short while, the environemnt should be up and ready. You can check the logs using

```
docker-compose logs -f
```

Apart from components mentioned above, also the Schema Registry UI and Kafka Manager have been started. You can reach them on

  * Schema Registry UI: <http://localhost:8002>
  * Kafka Manger: <http://localhost:9000>	
Tow topics `person-before` and `person-after` are automatically created using these commands:

```
kafka-topics --zookeeper zookeeper:2181 --create --topic person-before --replication-factor 1 --partitions 8
kafka-topics --zookeeper zookeeper:2181 --create --topic person-after --replication-factor 1 --partitions 8
```

## Avro Passthrough Stream Processor

This stream processor is implementing a generic pass-through behaviour for Kafka messages where both the key and the value are serialized avro objects. When starting the processor, the source and target topic has to be specified, as well as the Kafka cluster to connect to. 

It can be run with or without the "expire" behaviour, using the configuration parameter described below. 

The following options are available:

  * `-id` `--application-id`	ID of the stream processor application (mandatory)
  * `-b` `--bootsrap-sever`	Kafka broker(s) to connect to (mandatory)
  * `-s` `--source-topic`	Kafka topic to consume the messages from (mandatory)
  * `-t` `--target-topic`	Kafka topic to produce the messages to (mandatory)
  * `-sr` `--schema-registry`	URL to the confluent schema registry (mandatory)
  * `-e` `--expired-check`	flag signaling that message should be checked for expiration (optional)
  * `-v` `--verbose`	flag signaling that additional tracing output should be sent to stdout and stderr (opt

To run it from maven, you can execute

```
mvn exec:java -Dexec.args="--application-id person-pt-v1 --bootstrap-server localhost:9092 --source-topic person-before --target-topic person-after --schema-registry-url http://localhost:8081 --expired-check --verbose"
```

## Test Producer using Python
A Python script has been created for testing the behaviour based on the environment created above. It acts as the producer using the topic `person-before`.

### Prepare Python environment
To get the confluent python client, you first have to intall pip. 

```
sudo apt install python-pip
```

After that you can install the self-contained binaries of the Confluent Python client with avro support using

```
pip install confluent-kafka
pip install confluent-kafka[avro]
```

### Producing Messages

The test procuder sends `Person` messages, one at a time, using the following schema for the key

```
{
	"namespace": "my.test",
	"name": "PersonKey",
	"type": "record",
	"fields" : [
		 {
		   "name" : "id",
		   "type" : "string"
		 }
	]
}
```

and the following schema for the value portion of the message

```
{
	"namespace": "my.test",
	"name": "Person",
	"type": "record",
	   "fields" : [
		  {
		   "name" : "id",
		   "type" : "string"
		  },
		  {
		   "name" : "firstName",
		   "type" : "string"
		  },
		  {
		   "name" : "lastName",
		   "type" : "string"
		  }
	   ]
}
```

The test producer has to be called with the following five parameters, which define the value of the message:

* `brokers` - the list of broker(s) in the format `broker-1:9092,broker-2:9092`
* `schemaRegistryUrl` - the URL of the schema registry, in the format `http://localhost:8081`
* `timestamp` - the timestamp (in milli-sconds), use the following [link](https://currentmillis.com) to generate actual timestamps
* `id` - the ID of the person, will be used as the key
* `firstName` - the first name of the person
* `lastName` - the last name of the person

Here a sample call with all the necessary parameters

```
python producer.py localhost:9092 http://localhost:8081 1547628077671 10 Peter Muster
```

