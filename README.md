# amq-openshift
Deploying simple AMQ on OpenShift v4.

# Basic Instructions
- Create AMQ operator via Operators -> OperatorHub

- Create a ActiveMQArtemises broker instance as per amq-broker.yaml

- Generate SSL credentials as per [instructions](https://access.redhat.com/documentation/en-us/red_hat_amq/7.5/html-single/deploying_amq_broker_on_openshift/index#broker-operator-acceptor-configurationbroker-ocp) 

- Create Create ActiveMQArtemisAddress instance as per amq-address.yaml

- Create amqp service object in OpenShift as per amq-amqp-service.yaml

- Expose the amqp service and ensure that the tls termination is passthrough

- To send message:

```
mvn exec:java -Dexec.mainClass="com.redhat.demo.App" -D broker.url="amqps://amqp-amq-demo.apps.cluster-sgjj-9804.sgjj-9804.example.opentlc.com:443?transport.trustStoreLocation=client.ts&transport.trustStorePassword=password&transport.verifyHost=false" -Dsend.queue=MyQueue0 -Dsend.msg=sslHello -Dsend.mode=SEND
```

- To view received message:

```
mvn exec:java -Dexec.mainClass="com.redhat.demo.App" -D broker.url="amqps://amqp-amq-demo.apps.cluster-sgjj-9804.sgjj-9804.example.opentlc.com:443?transport.trustStoreLocation=client.ts&transport.trustStorePassword=password&transport.verifyHost=false" -Dsend.queue=MyQueue0  -Dsend.mode=RECV
```
