# KEDA Demo With Kafka Trigger

This repo contains everything to create a demo where you can scale Kafka consumers depending on the queue size on a Kafka Topic from scratch.

## Prerequisites
- A working Kubernetes cluster with enough resources 
- Helm for easy deployment of KEDA. KEDA offers different deployment options, so if you do not prefer Helm option, it can be istalled via Operator Hub or directly using Yaml files.

This demo leverages [Strimzi](https://strimzi.io)'s [Kafka Operator](https://github.com/strimzi/strimzi-kafka-operator) to run Kafka on Kubernetes.

This demo uses [kafka-producer](serdarkalayci/kafka-producer) and [kafka-consumer](serdarkalayci/kafka-consumer) images to interact with Kafka. The code for both images can be found [here](https://github.com/serdarkalayci/kafka-clients).

## Deployment
Follow the steps below to install the demo from scratch.

### Strimzi Kafka Operator Deployment
You can run [install-strimzi.sh](install-strimzi.sh) file, or copy and paste commands in the exact order to your terminal in order to install Smrimzi Kafka operator and deploy a single-broker and single-zookeeper Kafka cluster. The [cluster definition](kafka-cluster-ephemeral.yaml) is a copy of Strimzi's own [example](https://github.com/strimzi/strimzi-kafka-operator/blob/0.37.0/examples/kafka/kafka-ephemeral-single.yaml) with a tweak that won't allow auto topic creation. This is to prevent the producer from creating topics automatically, which creates topic with 1 partition and 1 replica by default. This in turn makes more than 1 consumers useless, and the whole demo becomes pointless.
When the cluster is in `Ready` state, you can create the topic by applying [topic.yaml](topic.yaml) file.
```sh
kubectl apply -f topic.yaml
```

## KEDA Operator Deployment
You can run [install-keda.sh](install-keda.sh) file, file, or copy and paste commands in the exact order to your terminal in order to install KEDA operator. If you prefer other deployment options than Helm, you can follow instructions [here](https://keda.sh/docs/2.5/deploy/).

## Kafka Producer Deployment
Kafka producer used in this demo exposes a web interface to interact with the end user. User can make requests to this endpoint to make producer produce messages on the topic a total number as a given URL parameter.

You can change the exposed port, Kafka Bootstrap address and Topic name using Env variables.

You can create a producer deployment and expose it using a LoadBalancer service. If you prefer you can use [port-forwarding](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) instead of a LoadBalancer service.

```sh
kubectl apply -f producer-deployment.yaml
kubectl apply -f producer-svc.yaml
```

## Kafka Consumer Deployment
Kafka consumer used in this demo leverages Consumer Group feature of Kafka. This allows Kafka to accept these consumers as dynamic consumers and dynamically assign partitions to these consumers.

You can change the Consumer Group, Sleep Duration, Kafka Bootstrap address and Topic name values using Env variables.

```sh
kubectl apply -f consumer-deployment.yaml
```

## ScaledObject
In order to autoscale our consumer deployment using KEDA depending on the queued message count on the Kafka Topic, we should create an object of type `scaledobjects.keda.sh`. When there are more than 5 messages in `keda-topic` topic, it will scale up the target object, which is the `kafka-consumer` deployment.

```sh
kubectl apply -f scaled-object.yaml
```

## Trying out
You can make requests to the Kafka Producer's endpoint including the message count you want it to produce to Kafka topic.
Because the consumers sleep between consuming each message, there'll be a congestion on the topic and KEDA will scale up the consumer deployment.

```sh
curl http://<EXTERNAL-IP-OF-PRODUCER-SVC>?count=100
```
