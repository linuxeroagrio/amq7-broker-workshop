# Lab 7. Clustered Brokers

## RHEL Create a pair of brokers

1. In the workstation machine stop `mybroker` broker

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
${AMQ_HOME}/instances/mybroker/bin/artemis-service stop

Gracefully Stopping artemis-service
```

2. Open a new terminal and create `broker1` broker

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
${AMQ_HOME}/bin/artemis create --user admin --password password --role admin --allow-anonymous y --clustered --host 127.0.0.1 --cluster-user clusterUser --cluster-password clusterPassword --max-hops 1 ${AMQ_HOME}/instances/broker1

Creating ActiveMQ Artemis instance at: /home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/broker1

Auto tuning journal ...
done! Your system can make 1.36 writes per millisecond, your journal-buffer-timeout will be 736000

You can now start the broker by executing:  

   "/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/broker1/bin/artemis" run

Or you can run the broker in the background using:

   "/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/broker1/bin/artemis-service" start
```

3. In a new terminal create `broker2` broker

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
${AMQ_HOME}/bin/artemis create  --user admin --password password --role admin --allow-anonymous y --clustered --host 127.0.0.1 --cluster-user clusterUser --cluster-password clusterPassword --max-hops 1 --port-offset 100 ${AMQ_HOME}/instances/broker2

Creating ActiveMQ Artemis instance at: /home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/broker2

Auto tuning journal ...
done! Your system can make 1.44 writes per millisecond, your journal-buffer-timeout will be 696000

You can now start the broker by executing:  

   "/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/broker2/bin/artemis" run

Or you can run the broker in the background using:

   "/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/broker2/bin/artemis-service" start
```

4. Start both brokers in each terminal/console

   >*Remember to allow traffic through the default ports. Check your firewall rules.*

```bash
# Terminal for broker1
${AMQ_HOME}/instances/broker1/bin/artemis run

# Terminal for broker2
${AMQ_HOME}/instances/broker2/bin/artemis run
```

After some initial negotiation you should see each broker log that a bridge has been created

```sh
2024-03-26 01:52:47,271 INFO  [org.apache.activemq.artemis.core.server] AMQ221027: Bridge ClusterConnectionBridge@344864ad [name=$.artemis.internal.sf.my-cluster.11ea0f25-eb35-11ee-8198-525400316e36, queue=QueueImpl[name=$.artemis.internal.sf.my-cluster.11ea0f25-eb35-11ee-8198-525400316e36, postOffice=PostOfficeImpl [server=ActiveMQServerImpl::name=127.0.0.1], temp=false]@65e4bb2a targetConnector=ServerLocatorImpl (identity=(Cluster-connection-bridge::ClusterConnectionBridge@344864ad [name=$.artemis.internal.sf.my-cluster.11ea0f25-eb35-11ee-8198-525400316e36, queue=QueueImpl[name=$.artemis.internal.sf.my-cluster.11ea0f25-eb35-11ee-8198-525400316e36, postOffice=PostOfficeImpl [server=ActiveMQServerImpl::name=127.0.0.1], temp=false]@65e4bb2a targetConnector=ServerLocatorImpl [initialConnectors=[TransportConfiguration(name=artemis, factory=org-apache-activemq-artemis-core-remoting-impl-netty-NettyConnectorFactory) ?port=61716&host=127-0-0-1], discoveryGroupConfiguration=null]]::ClusterConnectionImpl@918730310[nodeUUID=05b4c0f3-eb35-11ee-835a-525400316e36, connector=TransportConfiguration(name=artemis, factory=org-apache-activemq-artemis-core-remoting-impl-netty-NettyConnectorFactory) ?port=61616&host=127-0-0-1, address=, server=ActiveMQServerImpl::name=127.0.0.1])) [initialConnectors=[TransportConfiguration(name=artemis, factory=org-apache-activemq-artemis-core-remoting-impl-netty-NettyConnectorFactory) ?port=61716&host=127-0-0-1], discoveryGroupConfiguration=null]] is connected
```

5. Press `CTRL-C` in both terminals and start the brokers as a service.

```bash
# Terminal for broker1
${AMQ_HOME}/instances/broker1/bin/artemis-service start

# Terminal for broker2
${AMQ_HOME}/instances/broker2/bin/artemis-service start
```

## OCP Create a cluster of brokers

1. In the workstation machine create the `ActiveMQArtemis` instance with 2 replicas.

```bash
cat << EOF | oc replace -f -
apiVersion: broker.amq.io/v1beta1
kind: ActiveMQArtemis
metadata:
  name: ex-aao
  application: ex-aao-app
  namespace: amq-broker
spec:
  adminUser: admin
  adminPassword: admin
  console:
    expose: true
  deploymentPlan:
    image: placeholder
    jolokiaAgentEnabled: false
    journalType: nio
    managementRBACEnabled: true
    messageMigration: false
    persistenceEnabled: false
    requireLogin: true
    size: 2
EOF
```

2. Validate the creation of the broker instance and validate the logs

```bash
oc get pods -n amq-broker
NAME          READY   STATUS    RESTARTS   AGE
ex-aao-ss-0   1/1     Running   0          27m
ex-aao-ss-1   1/1     Running   0          28s

oc logs ex-aao-ss-0
2024-03-26 06:03:09,344 INFO  [org.apache.activemq.artemis.core.server] AMQ221027: Bridge ClusterConnectionBridge@5bc7274d [name=$.artemis.internal.sf.my-cluster.8394ea44-eb36-11ee-8073-0a580a840214, queue=QueueImpl[name=$.artemis.internal.sf.my-cluster.8394ea44-eb36-11ee-8073-0a580a840214, postOffice=PostOfficeImpl [server=ActiveMQServerImpl::name=amq-broker], temp=false]@17164286 targetConnector=ServerLocatorImpl (identity=(Cluster-connection-bridge::ClusterConnectionBridge@5bc7274d [name=$.artemis.internal.sf.my-cluster.8394ea44-eb36-11ee-8073-0a580a840214, queue=QueueImpl[name=$.artemis.internal.sf.my-cluster.8394ea44-eb36-11ee-8073-0a580a840214, postOffice=PostOfficeImpl [server=ActiveMQServerImpl::name=amq-broker], temp=false]@17164286 targetConnector=ServerLocatorImpl [initialConnectors=[TransportConfiguration(name=artemis, factory=org-apache-activemq-artemis-core-remoting-impl-netty-NettyConnectorFactory) ?port=61616&host=ex-aao-ss-1-ex-aao-hdls-svc-amq-broker-svc-cluster-local], discoveryGroupConfiguration=null]]::ClusterConnectionImpl@736714033[nodeUUID=ad75934b-eb32-11ee-b192-0a580a84020c, connector=TransportConfiguration(name=artemis, factory=org-apache-activemq-artemis-core-remoting-impl-netty-NettyConnectorFactory) ?port=61616&host=ex-aao-ss-0-ex-aao-hdls-svc-amq-broker-svc-cluster-local, address=, server=ActiveMQServerImpl::name=amq-broker])) [initialConnectors=[TransportConfiguration(name=artemis, factory=org-apache-activemq-artemis-core-remoting-impl-netty-NettyConnectorFactory) ?port=61616&host=ex-aao-ss-1-ex-aao-hdls-svc-amq-broker-svc-cluster-local], discoveryGroupConfiguration=null]] is connected
```

End of Lab 7.
