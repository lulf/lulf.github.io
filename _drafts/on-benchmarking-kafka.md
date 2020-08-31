---
layout: post
title:  "On benchmarking Kafka"
author: Ulf Lilleengen
categories: technical kubernetes kafka gitops
---

This is not going to be one of those blog posts about how great performance of messaging platform X is and how you should totally use it as your messaging platform. Instead, I would just like to share my experience using a tool I found to benchmark Apache Kafka. For a while now I have been playing around with [Strimzi](https://strimzi.io) and [Apache Kafka](https://kafka.apache.org), and I was looking for a way to benchmark Apache Kafka. I was initially using the kafka-producer-perf-test.sh and kafka-consumer-perf-test.sh found in the Apache Kafka distribution. Although these tools are useful to to a quick check of how well Apache Kafka performs, they require a lot of orchestration. I was looking for a tool that would

* support storing different test configurations;
* allow scaling producer and consumer clients across multiple hosts;
* store test results in a format like JSON; and
* (ideally) make it easy to graph results

Although it is possible to build something on top of the official scripts, I found the [OpenMessaging Benchmark Framework (OMB)](https://github.com/openmessaging/openmessaging-benchmark). OpenMessaging is part of the Linux Foundation and attempts to develop "vendor-neutral and language-independent industry standards and guidelines for the use of messaging and streaming applications across heterogeneous systems and platforms across industries, including finance, e-commerce, IoT and big-data; and an ecosystem of messaging components based on the new specification that can be reused across multiple projects and vendors.". You can read more about it at the [OpenMessaging Website](http://openmessaging.cloud/).

OMB is not directly related to the specification, but it provides test drivers for different messaging systems, one of them being Apache Kafka.

I found it useful that one can specify test properties as configuration files. For instance, the driver configuration for a particular Apache Kafka cluster can look like this:

---
name: Kafka
driverClass: io.openmessaging.benchmark.driver.kafka.KafkaBenchmarkDriver

replicationFactor: 3
topicConfig: |
  min.insync.replicas=2

commonConfig: |
  bootstrap.servers=bootstrap.my.cluster1:443

producerConfig: |
  acks=all
  linger.ms=1
  batch.size=131072

consumerConfig: |
  auto.offset.reset=earliest
  enable.auto.commit=false
---

Most of the settings are familiar if you already know Apache Kafka. A few settings that stand out is the `linger.ms` and `batch.size` configurations which allow you to configure how the producer uses batching to more efficiently transmit data.

The workload can be configured as follows:

---
name: My Workload
topics: 1
partitionsPerTopic: 1
messageSize: 100
payloadFile: "payload-100b.data"
subscriptionsPerTopic: 1
producersPerTopic: 1
consumerPerSubscription: 1
producerRate: 0
consumerBacklogSizeGB: 0
testDurationMinutes: 1
---

As with the driver, most settings are self-explanatory if you know Apache Kafka. OMB allow you to test for the max throughput of the system by setting `producerRate` to 0, which will cause the OMB producer to throttle its rate based on the `consumerBacklogSizeGB` setting. If consumers are lagging behind, the producer rate will go down. If there are no consumers lagging behind, the producer rate will be go up.

Having the driver and workload defined as configuration files allow you to store them in version control, and build automated performance tests based on that. Having them split into different files (drivers and workloads) allow you to more easily reuse test configurations.

You can also deploy OMB to Kuberenetes using the Helm chart. This model of deploying allow you to run the test clients on multiple hosts. Installing OMB using Helm will spin up a number of worker pods that can be assigned a consumer or producer by the coordinator. The coordinator accepts a list of hosts of the workers, and you can run it locally or on the cluster.

See, I promised that there would be no performance results. Performance depends on many variables that are different for different use cases, and it might not mean the same thing to every person. If you are tasked with benchmarking Kafka, I recommend trying this tool.
