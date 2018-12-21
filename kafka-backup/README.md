# Backup of Kafka Data

This project will investigate how a backup of Apache Kafka broker can be performed so that data can be recovered in case a topic is lost due to an error.

## Context

Situations we want to solve with this project:

* Someone has accidently deleted a topic
* Someone has produced wrong data into a topic

Situations we do not have to solve with this project:

* Disaster Recovery, if a whole data center is no longer available => use a DR setup of Kafka (either stretched cluster or mirroring)
* Consumer want to reconsume messages => use data retention on the topic which is large enough)

## Requirements

* Data should be backed-up to an external datastore (possibly S3) as a normal Kafka consumer (possibly using a Kafka connect connector). The data should be written in a way that it can efficiently and selectively be restored upon demand
* Data restore should be by topic, time (all, range)
* Data restore should be as fast as possible (using parallelism by partition)
* Data backup and restore should retain the order of the messages (by partition)
* Data retention on the topic should propagate to the backup data. If a segemnt is deleted from Kafka, then the corresponding data on the backup should be removed as well (after another configurable retention time).
* On Restore, an offset mapping index should be provided. We can store the offset of the original message (by topic, partition) with the backup. When restoring the data, a new offset will be created which can be mapped to the original offset. This would allow a consumer to move to the right offset by using the offset and after the restore continue with the last commited offset.  
* Solution should be as vender-neutral as possible
 
## References

1. [Surviving Data Loss](https://jobs.zalando.com/tech/blog/backing-up-kafka-zookeeper/index.html?gh_src=4n3gxh1) by Zalando
2. [Kafka Connect S3](https://github.com/spredfast/kafka-connect-s3) by Spredfast
