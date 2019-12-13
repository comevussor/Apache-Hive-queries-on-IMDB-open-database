# Streaming on Hadoop

We are dealing with :
- unbounded data sets (produced continuously)
- unbounded data processing (in time)
- low-latency, approximate and speculative results

Used tools :
- distributed messaging systems
  - publish/consume messages to/from queues
  - fault-tolerant
- distributed streaming processing engines
  - exactly-once fault-tolerant processing
  - aggregations, event-time windows

Architectures :
- Lambda architecture : separate batch processing (Hive) from streaming
- Kappa architecture : based on Kafka messaging system, build a sequence of queues with some processing in between each one.

Main opensource solutions : Apache Kafka and RabbitMQ. Both rely on publish-subscribe model. Kafka does not re-route messages (ssmart consumer )and is good for hard storage while RabbitMQ can re-route (dumb consumer) messages between queues but bad at hard storage.

## Apache Kafka

- Components are called brokers (servers).
- Queues are called topics that can be partitionned on different brokers.
- Producers and consumers can send to or receive from different brokers.
- Mid/long term storage
- Good integration with Hadoop ecosystem (Spark, Hive) : can query Kafka topics with Hive
    
Stream processing problematics
- event time vs processing time : difference is called skew
- we define time "windows" :
  - fixed : compute every 10 minutes
  - sliding windows : one 20' window every 10'
  - sessions : based on activity
- time marking : 
  - watermark : gives a deadline (processing time P) from which it is assumed that all data generated prior to an event time E have been observed, not always true, may discard some messages
  - trigger : time to materialize results
  
Dataflow model (Google model) :
- what results ar calulated ? aggregations
- where in event time are result calculated ? windows
- when in processing time are result materialized ?
- how do refinements of results relate ?

