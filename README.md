# MultithreadedKafkaConsumer
A multithreaded way on consuming kafka message.

The following project shows a generic code on how to consume kafka message in a multi threaded way. 
The code is in Java using Spring framework together with the Apache Kafka library.

The design is to separate the code for consuming Kafka message and have a way to distribute the load 
of the kafka message into threads. One benefit of the boilerplate code is to allow unlimited way to separate
the task via configuration file.

*A limitation of the design is it does not yet support multi partitioned Kafka messages, this will be done in the future*

The kafka message shall have a key that contains specific key that can be spread to several processing 
thread using regular expression. Let us assume that the key of the Kafka message is a UUID, which means 
that the key will end in a hexadecimal value. This limits our possible value to 16 possible characters (0-F).me

The most basic configuration of the system is to accept it one is to one. Let us call this configuration as '1-1'
```yml
multikafkaconsumer:
 consumerSettings:
  # Consumes all Kafka messages regardless of the key value
  -
   consumerKey: C1
   consumeFilter: *
 processorSettings:
  # Processes Kafka messages regardless of the key value
  -
   processorKey: P1
   processFilter: *
   acceptFromConsumer: *
```

Now assume that the uuid has biases and there are a lot of message and we want to spread the load to two thread 
for the consumer. We would separate the consuming of Kafka messages with keys ending with 0-7 and 8-F. 
More over the processing of data from keys with UUID that ends with 0-3 and 4-7 takes more time, so we would like
to spread the processing to separate threads.

The configuration file is as follows:
```yml
multikafkaconsumer:
 consumerSettings:
  # Consumes Kafka messages with key that ends with 0 to 7
  -
   consumerKey: C1
   consumeFilter: [0-7]$
  # Consumes Kafka messages with key that ends with 8 to D
  -
   consumerKey: C2
   consumeFilter: [89ABCDabcd]$
 processorSettings:
  # Processes Kafka messages with key that ends with 0 to 3 from consumer C1
  -
   processorKey: P1
   processFilter: [0-3]$
   acceptFromConsumer: C1
  # Processes Kafka messages with key that ends with 8 to B from consumer C2
  -
   processorKey: P2
   processFilter: [89ABab]$
   acceptFromConsumer: C2
  # Processes Kafka messages with key that ends with 4 to 7 and C to F from both consumers
  -
   processorKey: P3
   processFilter: [4-7]|[CDEFcdef]$
   acceptFromConsumer: *
```
