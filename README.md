# mirrormaker
Terminology
For the purposes of this post, we’ll refer to the two Kafka clusters as:

“origin” – your existing Kafka cluster that you are migrating from
“target” – your new Kafka cluster that you are migrating to
The instructions here use the Strimzi Operator as a convenient way to configure and run MirrorMaker 2, but neither the “origin” or “target” Kafka clusters need to be managed by the Strimzi Operator for this to work.

We’ll be running MirrorMaker 2 in a namespace called “migration”.


step 1 run kafkauser on Origin cluster
step 2 run secret on Origin cluster

If your “origin” cluster is managed by Strimzi, 
you will be able to find this in a Secret called something like origin-cluster-ca-cert in the namespace where your Kafka cluster is running.
step 3 run tls-secret on Origin cluster -->--> add the ca.crt from the secret!

step 4 run target-user on Target cluster
step 5 run credentials-target on Target cluster

If your “target” cluster is managed by Strimzi, 
you will be able to find this in a Secret called something like target-cluster-ca-cert in the namespace where your Kafka cluster is running.
step 6 rum  tls-secret-target --> add the ca.crt from the secret!

step 7 review and customize the file KafkaMirrorMaker
step 8 one you read and add the values in KafakamirrorMaker file  run it.

Enjoy the data replication across cluster, DC, etc

do not forget to thank https://dalelane.co.uk/blog/?p=4303

if this helped do not forget to contribute in case you did an improvement!!
