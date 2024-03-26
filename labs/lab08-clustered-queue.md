# Lab 8. Clustered Queue

## RHEL Create a clustered queue

To have a clustered queue it must be defined in both brokers.

1. Open two terminals and add an anycast queue configuration to both brokers.

```bash
# For broker1 terminal
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
vim ${AMQ_HOME}/instances/broker1/etc/broker.xml

# For broker2 terminal
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
vim ${AMQ_HOME}/instances/broker2/etc/broker.xml
```

```XML
<addresses>
...
...
...
   <address name="clusteredQueue">
      <anycast>
         <queue name="clusteredQueue"/>
      </anycast>
   </address>
</addresses>
```

2. Restart the brokers

```bash
# For broker1 terminal
${AMQ_HOME}/instances/broker1/bin/artemis-service restart
Restarting artemis-service
artemis-service is now running (6031)

# For broker2 terminal
${AMQ_HOME}/instances/broker2/bin/artemis-service restart
Restarting artemis-service
artemis-service is now running (6186)
```

## Run a consumer connected to each broker

1. In `broker1` terminal

```bash
${AMQ_HOME}/instances/broker1/bin/artemis consumer --message-count 5 --destination queue://clusteredQueue
```

2. In `broker2` terminal

```bash
${AMQ_HOME}/instances/broker2/bin/artemis consumer --message-count 5 --url tcp://localhost:61716 --destination queue://clusteredQueue
```

3. Now in a third terminal run the producer connected to `broker1` and send 10 messages

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
${AMQ_HOME}/instances/broker1/bin/artemis producer --message-count 10 --destination queue://clusteredQueue
```

   >*You will see each consumer receives 5 messages in a round robin fashion*

```bash
${AMQ_HOME}/instances/broker1/bin/artemis consumer --message-count 5 --destination queue://clusteredQueue
Connection brokerURL = tcp://127.0.0.1:61616
Consumer:: filter = null
Consumer ActiveMQQueue[clusteredQueue], thread=0 wait until 5 messages are consumed
Consumer ActiveMQQueue[clusteredQueue], thread=0 Consumed: 5 messages
Consumer ActiveMQQueue[clusteredQueue], thread=0 Elapsed time in second : 120 s
Consumer ActiveMQQueue[clusteredQueue], thread=0 Elapsed time in milli second : 120669 milli seconds
Consumer ActiveMQQueue[clusteredQueue], thread=0 Consumed: 5 messages
Consumer ActiveMQQueue[clusteredQueue], thread=0 Consumer thread finished

${AMQ_HOME}/instances/broker2/bin/artemis-service start
Starting artemis-service
artemis-service is now running (5807)
[lab-user@bastion ~]$ ${AMQ_HOME}/instances/broker2/bin/artemis consumer --message-count 5 --url tcp://localhost:61716 --destination queue://clusteredQueue
Connection brokerURL = tcp://localhost:61716
Consumer:: filter = null
Consumer ActiveMQQueue[clusteredQueue], thread=0 wait until 5 messages are consumed
Consumer ActiveMQQueue[clusteredQueue], thread=0 Consumed: 5 messages
Consumer ActiveMQQueue[clusteredQueue], thread=0 Elapsed time in second : 110 s
Consumer ActiveMQQueue[clusteredQueue], thread=0 Elapsed time in milli second : 110022 milli seconds
Consumer ActiveMQQueue[clusteredQueue], thread=0 Consumed: 5 messages
Consumer ActiveMQQueue[clusteredQueue], thread=0 Consumer thread finished

${AMQ_HOME}/instances/broker1/bin/artemis producer --message-count 10 --destination queue://clusteredQueue
Connection brokerURL = tcp://127.0.0.1:61616
Producer ActiveMQQueue[clusteredQueue], thread=0 Started to calculate elapsed time ...

Producer ActiveMQQueue[clusteredQueue], thread=0 Produced: 10 messages
Producer ActiveMQQueue[clusteredQueue], thread=0 Elapsed time in second : 0 s
Producer ActiveMQQueue[clusteredQueue], thread=0 Elapsed time in milli second : 99 milli seconds
```

## Run a disconnected consumer

1. Run the broker2 consumer first

```bash
${AMQ_HOME}/instances/broker2/bin/artemis consumer --message-count 10 --url tcp://localhost:61716 --destination queue://clusteredQueue
```

2. Now run the producer again

```bash
${AMQ_HOME}/instances/broker1/bin/artemis producer --message-count 10 --destination queue://clusteredQueue
```

3. Run the broker1 consumer next

```bash
${AMQ_HOME}/instances/broker1/bin/artemis consumer --message-count 5 --destination queue://clusteredQueue
```
   >*You will see that only 1 consumer received the messages as they were load balanced on demand*

Press `CTRL+C`in the three terminals

## Message load balancing strategy

1. Open two terminals and update the load balancing to strict in the `etc/broker.xml` file for both brokers.

```bash
# For broker1 terminal
vim ${AMQ_HOME}/instances/broker1/etc/broker.xml

# For broker2 terminal
vim ${AMQ_HOME}/instances/broker2/etc/broker.xml
```

```XML
<cluster-connections>
      <cluster-connection name="my-cluster">
         <connector-ref>artemis</connector-ref>
         <message-load-balancing>STRICT</message-load-balancing>
         <max-hops>1</max-hops>
         <discovery-group-ref discovery-group-name="dg-group1"/>
      </cluster-connection>
</cluster-connections>
```

2. Restart the brokers and run again the previous exercise

```bash
# For broker1 terminal
${AMQ_HOME}/instances/broker1/bin/artemis-service restart
Restarting artemis-service
artemis-service is now running (6031)

# For broker2 terminal
${AMQ_HOME}/instances/broker2/bin/artemis-service restart
Restarting artemis-service
artemis-service is now running (6186)
```

   >*You will notice that both the consumers receive messages even when disconnected*

End of Lab 8.
