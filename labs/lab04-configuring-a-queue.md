# Lab 4. Configuring a Queue

## RHEL Creating a Queue using the CLI

1. In the workstation machine, open the `${AMQ_HOME}/etc/broker.xml` file with your favorite text editor.

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
vim ${AMQ_HOME}/instances/mybroker/etc/broker.xml
```

2. Locate the `<addresses></addresses>` section and add an `anycast` type address with a queue.
```XML
<addresses>
...
...
...
   <address name="testQueue">
      <anycast>
         <queue name="testQueue"/>
      </anycast>
   </address>
</addresses>
```

3. Start the broker or restart in case it’s already running.

```bash
${AMQ_HOME}/instances/mybroker/bin/artemis-service restart
Restarting artemis-service
artemis-service is now running (3590)
```

4. Test the new created queue using running the following command in two (2) diferent terminal consoles

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
${AMQ_HOME}/instances/mybroker/bin/artemis consumer --destination queue://testQueue --message-count 5
```

5. Run the producer in a third terminal/console

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
${AMQ_HOME}/instances/mybroker/bin/artemis producer --destination queue://testQueue --message-count 5
```

```bash
#Consumer Output
${AMQ_HOME}/instances/mybroker/bin/artemis consumer --destination queue://testQueue --message-count 5

Connection brokerURL = tcp://localhost:61616
Consumer:: filter = null
Consumer ActiveMQQueue[testQueue], thread=0 wait until 5 messages are consumed
Consumer ActiveMQQueue[testQueue], thread=0 Consumed: 5 messages
Consumer ActiveMQQueue[testQueue], thread=0 Elapsed time in second : 80 s
Consumer ActiveMQQueue[testQueue], thread=0 Elapsed time in milli second : 80330 milli seconds
Consumer ActiveMQQueue[testQueue], thread=0 Consumed: 5 messages
Consumer ActiveMQQueue[testQueue], thread=0 Consumer thread finished

#Producer Output
${AMQ_HOME}/instances/mybroker/bin/artemis producer --destination queue://testQueue --message-count 5

Connection brokerURL = tcp://localhost:61616
Producer ActiveMQQueue[testQueue], thread=0 Started to calculate elapsed time ...

Producer ActiveMQQueue[testQueue], thread=0 Produced: 5 messages
Producer ActiveMQQueue[testQueue], thread=0 Elapsed time in second : 0 s
Producer ActiveMQQueue[testQueue], thread=0 Elapsed time in milli second : 97 milli seconds
```

6. Exit from the consumer and producer terminal sessions.

## OCP Creating a Queue

1. In the workstation machine, create an `ActiveMQArtemisAddress` resource

```bash
cat << EOF | oc apply -f -
kind: ActiveMQArtemisAddress
apiVersion: broker.amq.io/v1beta1
metadata:
  name: testqueue-address
  namespace: amq-broker
spec:
  addressName: testQueue
  queueName: testQueue
  routingType: anycast
EOF

oc get ActiveMQArtemisAddress testqueue-address -n amq-broker

NAME                AGE
testqueue-address   9s
```

2. Run the artemis command to consume from the address

```bash
oc exec -ti ex-aao-ss-0 -c ex-aao-container -n amq-broker -- amq-broker/bin/artemis consumer --url tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616 --user admin --password admin --destination queue://testQueue --message-count 5
```

3. In another terminal/console do the same but this time to produce messages to the same address

```bash
oc exec -ti ex-aao-ss-0 -c ex-aao-container -n amq-broker -- amq-broker/bin/artemis producer --url tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616 --user admin --password admin --destination queue://testQueue --message-count 5
```

```bash
#Consumer Output
oc exec -ti ex-aao-ss-0 -c ex-aao-container -n amq-broker -- amq-broker/bin/artemis consumer --url tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616 --user admin --password admin --destination queue://testQueue --message-count 5

Connection brokerURL = tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616
Consumer:: filter = null
Consumer ActiveMQQueue[testQueue], thread=0 wait until 5 messages are consumed
Consumer ActiveMQQueue[testQueue], thread=0 Consumed: 5 messages
Consumer ActiveMQQueue[testQueue], thread=0 Elapsed time in second : 23 s
Consumer ActiveMQQueue[testQueue], thread=0 Elapsed time in milli second : 23802 milli seconds
Consumer ActiveMQQueue[testQueue], thread=0 Consumed: 5 messages
Consumer ActiveMQQueue[testQueue], thread=0 Consumer thread finished

#Producer Output
oc exec -ti ex-aao-ss-0 -c ex-aao-container -n amq-broker -- amq-broker/bin/artemis producer --url tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616 --user admin --password admin --destination queue://testQueue --message-count 5

Connection brokerURL = tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616
Producer ActiveMQQueue[testQueue], thread=0 Started to calculate elapsed time ...

Producer ActiveMQQueue[testQueue], thread=0 Produced: 5 messages
Producer ActiveMQQueue[testQueue], thread=0 Elapsed time in second : 0 s
Producer ActiveMQQueue[testQueue], thread=0 Elapsed time in milli second : 27 milli seconds
```

4. Close the addditional terminals

End of Lab 4.
