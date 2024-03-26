# Lab 5. Configuring a Topic

## RHEL Creating a Topic using the CLI

1. In the workstation machine, open the `${AMQ_HOME}/etc/broker.xml` file with your favorite text editor.

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
vim ${AMQ_HOME}/instances/mybroker/etc/broker.xml
```

2. Locate the `<addresses></addresses>` section and add an `multicast` type address with a topic.
```XML
<addresses>
...
...
...
   <address name="testTopic">
      <multicast>
         <queue name="testTopic"/>
      </multicast>
   </address>
</addresses>
```

3. Start the broker or restart in case it’s already running.

```bash
${AMQ_HOME}/instances/mybroker/bin/artemis-service restart
Restarting artemis-service
artemis-service is now running (4050)
```

4. Test the new created queue using running the following command in two (2) diferent terminal consoles

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
${AMQ_HOME}/instances/mybroker/bin/artemis consumer --destination topic://testTopic --message-count 10
```

5. Run the producer in a third terminal/console

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
${AMQ_HOME}/instances/mybroker/bin/artemis producer --destination topic://testTopic --message-count 10
```

```bash
#Consumer Output
${AMQ_HOME}/instances/mybroker/bin/artemis consumer --destination topic://testTopic --message-count 10

Connection brokerURL = tcp://localhost:61616
Consumer:: filter = null
Consumer ActiveMQTopic[testTopic], thread=0 wait until 10 messages are consumed
Consumer ActiveMQTopic[testTopic], thread=0 Consumed: 10 messages
Consumer ActiveMQTopic[testTopic], thread=0 Elapsed time in second : 52 s
Consumer ActiveMQTopic[testTopic], thread=0 Elapsed time in milli second : 52389 milli seconds
Consumer ActiveMQTopic[testTopic], thread=0 Consumed: 10 messages
Consumer ActiveMQTopic[testTopic], thread=0 Consumer thread finished

#Producer Output
${AMQ_HOME}/instances/mybroker/bin/artemis producer --destination topic://testTopic --message-count 10

Connection brokerURL = tcp://localhost:61616
Producer ActiveMQTopic[testTopic], thread=0 Started to calculate elapsed time ...

Producer ActiveMQTopic[testTopic], thread=0 Produced: 10 messages
Producer ActiveMQTopic[testTopic], thread=0 Elapsed time in second : 0 s
Producer ActiveMQTopic[testTopic], thread=0 Elapsed time in milli second : 97 milli seconds
```

6. Exit from the consumer and producer terminal sessions.

## OCP Creating a Topic

1. In the workstation machine, create an `ActiveMQArtemisAddress` resource

```bash
cat << EOF | oc apply -f -
kind: ActiveMQArtemisAddress
apiVersion: broker.amq.io/v1beta1
metadata:
  name: testtopic-address
  namespace: amq-broker
spec:
  addressName: testTopic
  queueName: testTopic
  routingType: multicast
EOF

oc get ActiveMQArtemisAddress testtopic-address -n amq-broker

NAME                AGE
testtopic-address   6s
```

2. Run the artemis command to consume from address

```bash
oc exec -ti ex-aao-ss-0 -c ex-aao-container -n amq-broker -- amq-broker/bin/artemis consumer --url tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616 --user admin --password admin --destination topic://testTopic --message-count 10
```

3. In another terminal/console do the same but this time to produce messages to the same address

```bash
oc exec -ti ex-aao-ss-0 -c ex-aao-container -n amq-broker -- amq-broker/bin/artemis producer --url tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616 --user admin --password admin --destination topic://testTopic --message-count 10
```

```bash
#Consumer Output
oc exec -ti ex-aao-ss-0 -c ex-aao-container -n amq-broker -- amq-broker/bin/artemis consumer --url tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616 --user admin --password admin --destination topic://testTopic --message-count 10

Connection brokerURL = tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616
Consumer:: filter = null
Consumer ActiveMQTopic[testTopic], thread=0 wait until 10 messages are consumed
Consumer ActiveMQTopic[testTopic], thread=0 Consumed: 10 messages
Consumer ActiveMQTopic[testTopic], thread=0 Elapsed time in second : 10 s
Consumer ActiveMQTopic[testTopic], thread=0 Elapsed time in milli second : 10734 milli seconds
Consumer ActiveMQTopic[testTopic], thread=0 Consumed: 10 messages
Consumer ActiveMQTopic[testTopic], thread=0 Consumer thread finished

#Producer Output
oc exec -ti ex-aao-ss-0 -c ex-aao-container -n amq-broker -- amq-broker/bin/artemis producer --url tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616 --user admin --password admin --destination topic://testTopic --message-count 10
Connection brokerURL = tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616
Producer ActiveMQTopic[testTopic], thread=0 Started to calculate elapsed time ...

Producer ActiveMQTopic[testTopic], thread=0 Produced: 10 messages
Producer ActiveMQTopic[testTopic], thread=0 Elapsed time in second : 0 s
Producer ActiveMQTopic[testTopic], thread=0 Elapsed time in milli second : 47 milli seconds
```

4. Close the addditional terminals

End of Lab 5.
