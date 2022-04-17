
# Ultimate MirrorMaker2 Strimzi Tutorial

For Months I struggle trying to make fully work MirrorMaker. Specially if TLS is implemented. The problem is that there is not a full tutorial, on how to install it and how to make it work. You need to do tons of little independent researches and glue them all together (in the right order). So it becomes an expert exercise, complicated and with tons of frustration on the way to learn by trail an error. Overall is not hard or complex, is just there is not enough information out there in one single place to make it work step by step.... Until now!

## Assumptions:

- You have strimzi installed and working (minimum in two clusters). If you I do recommend you to go to their official webpage and follow their installation tutorials, join their slack and say thanks to Jakub. He is a really nice guy always helping.
- You have two different Kafkas running in the same or in a different kubernetes but over all this exercise is a how to do a data-sync between two different kafka clusters.
- You have good knowledge of kubernetes. if not this is the wrong place to start and I do recommend you to start a course in kubernetes, then comeback.

## Terminology
For the purposes of this tutorial, we’ll refer to the two Kafka clusters as:

“origin” – your existing Kafka cluster that you are migrating from

“target” – your new Kafka cluster that you are migrating to

The instructions here use the Strimzi Operator as a convenient way to configure and run MirrorMaker 2, but neither the “origin” or “target” Kafka clusters need to be managed by the Strimzi Operator for this to work.

We’ll be running MirrorMaker 2 in a namespace called “migration”.

## Steps

1. Before you start, you need to obtain credentials and TLS certificates for both of your Kafka clusters:

    Credentials from the “origin” cluster
    TLS certificate from the “origin” cluster
    Credentials from the “target” cluster
    TLS certificate from the “target” cluster

Credentials from the “origin” cluster

If your “origin” cluster requires authentication, you will need to create credentials for MirrorMaker 2 to use.

If your “origin” cluster is managed by Strimzi, you could do this by creating a KafkaUser resource something like this 

Look at the Kafka-user-origin.yaml file
apiVersion: kafka.strimzi.io/v1beta1
```
kind: KafkaUser
metadata:
  name: mm2-credentials
  labels:
    strimzi.io/cluster: origin
  namespace: origin-ns
spec:
  authentication:
    type: scram-sha-512
  authorization:
    acls:
      - host: '*'
        operation: Read
        resource:
          name: '*'
          patternType: literal
          type: topic
      - host: '*'
        operation: Describe
        resource:
          name: '*'
          patternType: literal
          type: topic
      - host: '*'
        operation: DescribeConfigs
        resource:
          name: '*'
          patternType: literal
          type: topic
      - host: '*'
        operation: Create
        resource:
          type: cluster
      - host: '*'
        operation: Read
        resource:
          type: cluster
      - host: '*'
        operation: Describe
        resource:
          type: cluster
      - host: '*'
        operation: Write
        resource:
          name: '*'
          patternType: literal
          type: topic
      - host: '*'
        operation: Describe
        resource:
          name: '*'
          patternType: literal
          type: group
      - host: '*'
        operation: Read
        resource:
          name: '*'
          patternType: literal
          type: group
    type: simple
```
This will create a new secret with your credentials in the namespace where your “origin” Kafka cluster is running. Copy these into a secret in a namespace on the Kubernetes cluster where you want to run MirrorMaker 2.

2. Look at kafka-password-origin.yaml and run it in the original cluster

These credentials will allow MirrorMaker to find and consume from all of the topics on the “origin” cluster. You will need to do something similar using the type of Kafka cluster you are running. However you create your credentials, you should create a Secret similar to the one above.

```
apiVersion: v1
kind: Secret
metadata:
  name: origin-cluster-credentials
  namespace: migration
data:
  user.crt
  user.key
``` 

3. If your “origin” cluster requires TLS, you may need to obtain a CA cert for MirrorMaker 2 to use.

If your “origin” cluster is managed by Strimzi, you will be able to find this in a Secret called something like CLUSTERNAME-cluster-ca-cert in the namespace where your Kafka cluster is running. Now is time to create a secret wit the ca.crt:
```
kind: Secret
metadata:
  name: origin-cluster-ca-cert
  namespace: migration
data:
  ca.crt:
```

## Now you have all the secrets and users in Target
   Next steps are the same but now in Origin. So it should be very similar 3 steps.

1. create user in Target
2. create a secret from a user in Target
3. create a secret from the internal CLUSTERNAME-ca-cert in target

You should/could use the example files I have in this same tutorial.
**
NOTE: you do not need to create a user. I do not use a user only authentication via TLS and I do steps 1 & 3. My connections works and is fully encrypted and secure between my two clusters!**

## Once I have all the users and secrets in both clusters

So we have all the secrets and it is time to decide where you are going to run MMK. It can run in a 3rd cluster or it can ran on the target, even on the origin cluster is a service independent that where you run it it should work as long as you give the right certs.


