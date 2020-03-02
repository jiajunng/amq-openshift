# amq-openshift-demo
Deploying simple AMQ v7.5 on OpenShift v4.

# Basic Instructions
- Create AMQ operator via Operators -> OperatorHub

- Create a ActiveMQArtemises broker instance as per amq-broker.yaml

```
apiVersion: broker.amq.io/v2alpha1
kind: ActiveMQArtemis
metadata:
  name: ex-aao
  namespace: amq-demo
spec:
  acceptors:
    - name: amqp
      needClientAuth: false
      port: 5671
      protocols: amqp
      sslEnabled: true
      sslSecret: ex-aao-amqp-secret
      verifyHost: false
  deploymentPlan:
    image: 'registry.redhat.io/amq7/amq-broker:7.5'
    size: 1
```

- Generate SSL credentials as per [official docs](https://access.redhat.com/documentation/en-us/red_hat_amq/7.5/html-single/deploying_amq_broker_on_openshift/index#broker-operator-acceptor-configurationbroker-ocp):

```
# keytool -genkey -alias broker -keyalg RSA -keystore broker.ks
# keytool -export -alias broker -keystore broker.ks -file broker_cert
# keytool -genkey -alias client -keyalg RSA -keystore client.ks
# keytool -import -alias broker -keystore client.ts -file broker_cert
# oc secrets new amq-app-secret broker.ks client.ts
# oc secrets add sa/amq-broker-operator secret/amq-app-secret
```

- Create Create ActiveMQArtemisAddress instance as per amq-address.yaml:

```
apiVersion: broker.amq.io/v2alpha1
kind: ActiveMQArtemisAddress
metadata:
  name: ex-aaoaddress
  namespace: amq-demo
spec:
  addressName: myAddress0
  queueName: myQueue0
  routingType: anycast
```

- Create amqp service object in OpenShift as per amq-amqp-service.yaml

```
kind: Service
apiVersion: v1
metadata:
  name: amqp
  namespace: amq-demo
spec:
  ports:
    - name: amqp
      protocol: TCP
      port: 5671
      targetPort: 5671
  selector:
    ActiveMQArtemis: ex-aao
    application: ex-aao-app
  clusterIP: None
  type: ClusterIP
  sessionAffinity: None
  publishNotReadyAddresses: true
```

- Expose the amqp service and ensure that the tls termination is passthrough

- Simple Java client can be found [https://github.com/jiajunng/simple-amq-client.git](https://github.com/jiajunng/simple-amq-client.git)

- To send message (Edit the OpenShift route and the location of client.ts):

```
mvn exec:java -Dexec.mainClass="com.redhat.demo.App" -D broker.url="amqps://amqp-amq-demo.apps.cluster-sgjj-9804.sgjj-9804.example.opentlc.com:443?transport.trustStoreLocation=client.ts&transport.trustStorePassword=password&transport.verifyHost=false" -Dsend.queue=MyQueue0 -Dsend.msg=sslHello -Dsend.mode=SEND
```

- To view received message (Edit the OpenShift route and the location of client.ts):

```
mvn exec:java -Dexec.mainClass="com.redhat.demo.App" -D broker.url="amqps://amqp-amq-demo.apps.cluster-sgjj-9804.sgjj-9804.example.opentlc.com:443?transport.trustStoreLocation=client.ts&transport.trustStorePassword=password&transport.verifyHost=false" -Dsend.queue=MyQueue0  -Dsend.mode=RECV
```
