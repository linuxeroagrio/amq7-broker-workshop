# Lab 9. Shared Store

## Create a high available broker with a shared store using the CLI

1. In Workstation machoine stop the `broker1` and `broker2` brokers

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
${AMQ_HOME}/instances/broker1/bin/artemis-service stop

Gracefully Stopping artemis-service

${AMQ_HOME}/instances/broker2/bin/artemis-service stop

Gracefully Stopping artemis-service
```

2. Open a new terminal and create a live `libeBroker` broker

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
${AMQ_HOME}/bin/artemis create --shared-store --failover-on-shutdown --data ${AMQ_HOME}/instances/liveBroker/data --user admin --password password --role admin --allow-anonymous y --clustered --host 127.0.0.1 --cluster-user clusterUser --cluster-password clusterPassword --max-hops 1 ${AMQ_HOME}/instances/liveBroker

Creating ActiveMQ Artemis instance at: /home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/liveBroker

Auto tuning journal ...
done! Your system can make 1.25 writes per millisecond, your journal-buffer-timeout will be 800000

You can now start the broker by executing:  

   "/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/liveBroker/bin/artemis" run

Or you can run the broker in the background using:

   "/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/liveBroker/bin/artemis-service" start
```

3. In a new terminal create backup `backupBroker` broker

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
${AMQ_HOME}/bin/artemis create --shared-store --failover-on-shutdown --slave --data ${AMQ_HOME}/instances/liveBroker/data --user admin --password password --role admin --allow-anonymous y --clustered --host 127.0.0.1 --cluster-user clusterUser --cluster-password clusterPassword --max-hops 1 --port-offset 100 ${AMQ_HOME}/instances/backupBroker

Creating ActiveMQ Artemis instance at: /home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/backupBroker

Auto tuning journal ...
done! Your system can make 1.35 writes per millisecond, your journal-buffer-timeout will be 740000

You can now start the broker by executing:  

   "/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/backupBroker/bin/artemis" run

Or you can run the broker in the background using:

   "/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/backupBroker/bin/artemis-service" start
```

4. Add an `anycast` queue configuration to both brokers.

```bash
# For liveBroker terminal
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
vim ${AMQ_HOME}/instances/liveBroker/etc/broker.xml

# For backupBroker terminal
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
vim ${AMQ_HOME}/instances/backupBroker/etc/broker.xml
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

5. Start the brokers, first the `liveBroker`, then the `backupBroker`.

```bash
# Terminal for broker1
${AMQ_HOME}/instances/liveBroker/bin/artemis run

# Terminal for broker2
${AMQ_HOME}/instances/backupBroker/bin/artemis run
```

>*You should see the backup broker announce itself as a **backup** in the logs.*

```bash
2024-03-26 02:50:32,801 INFO  [org.apache.activemq.artemis.core.server] AMQ221031: backup announced
```

6. Terminate the `liveBroker` by pressing `CTRL+C` 

   >*The `backupBroker` should start as live*

```bash
2024-03-26 02:52:22,182 INFO  [org.apache.activemq.artemis.core.server] AMQ221010: Backup Server is now live
```

7. Restart the `liveBroker`

```bash
# Terminal for broker1
${AMQ_HOME}/instances/liveBroker/bin/artemis run
```

   >*The `backupBroker` will automatically shutdown  and the `liveBroker` restart*

```bash
2024-03-26 02:53:27,323 INFO  [org.apache.activemq.artemis.core.server] AMQ221008: live server wants to restart, restarting server in backup
```

End of Lab 9.
