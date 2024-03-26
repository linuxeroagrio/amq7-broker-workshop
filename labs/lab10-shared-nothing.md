# Lab 10. Shared Nothing (replicated)

## Create a high available broker with a shared nothing (replicated) using the CLI

1. In Workstation machine, create a live `repLiveBroker` broker

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
${AMQ_HOME}/bin/artemis create --replicated --failover-on-shutdown  --user admin --password password --role admin --allow-anonymous y --clustered --host 127.0.0.1 --cluster-user clusterUser --cluster-password clusterPassword  --max-hops 1 ${AMQ_HOME}/instances/repLiveBroker

Creating ActiveMQ Artemis instance at: /home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/repLiveBroker

Auto tuning journal ...
done! Your system can make 1.11 writes per millisecond, your journal-buffer-timeout will be 900000

You can now start the broker by executing:  

   "/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/repLiveBroker/bin/artemis" run

Or you can run the broker in the background using:

   "/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/repLiveBroker/bin/artemis-service" start
```

2. In a new terminal create backup `repBackupBroker` broker

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
${AMQ_HOME}/bin/artemis create --replicated --failover-on-shutdown --slave --user admin --password password --role admin --allow-anonymous y --clustered --host 127.0.0.1 --cluster-user clusterUser --cluster-password clusterPassword  --max-hops 1 --port-offset 100 ${AMQ_HOME}/instances/repBackupBroker

Creating ActiveMQ Artemis instance at: /home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/repBackupBroker

Auto tuning journal ...
done! Your system can make 1.46 writes per millisecond, your journal-buffer-timeout will be 684000

You can now start the broker by executing:  

   "/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/repBackupBroker/bin/artemis" run

Or you can run the broker in the background using:

   "/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/repBackupBroker/bin/artemis-service" start
```

3. Add an `anycast` queue configuration to both brokers.

```bash
# For repLiveBroker terminal
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
vim ${AMQ_HOME}/instances/repLiveBroker/etc/broker.xml

# For repBackupBroker terminal
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
vim ${AMQ_HOME}/instances/repBackupBroker/etc/broker.xml
```

```XML
<addresses>
...
...
...
  <address name="haQueue">
    <anycast>
           <queue name="haQueue"/>
    </anycast>
  </address>
</addresses>
```

4. Start the brokers, first the `repLiveBroker`, then the `repBackupBroker`

```bash
# Terminal for repLiveBroker
${AMQ_HOME}/instances/repLiveBroker/bin/artemis run

# Terminal for repBackupBroker
${AMQ_HOME}/instances/repBackupBroker/bin/artemis run
```

>*You should see the backup broker announce itself as a **backup** in the logs and the journal being replicated.*

```bash
2024-03-26 03:06:06,380 INFO  [org.apache.activemq.artemis.core.server] AMQ221109: Apache ActiveMQ Artemis Backup Server version 2.28.0.redhat-00003 [null] started, waiting live to fail before it gets active
2024-03-26 03:06:06,906 INFO  [org.apache.activemq.artemis.core.server] AMQ221024: Backup server ActiveMQServerImpl::name=127.0.0.1 is synchronized with live server, nodeID=4741de20-eb3f-11ee-bb4d-525400316e36.
2024-03-26 03:06:11,169 INFO  [org.apache.activemq.artemis.core.server] AMQ221031: backup announced
```

5. Terminate the repLiveBroker by pressing `CTRL+C` 

>*The `repBackupBroker` should start as live.*

```bash
2024-03-26 03:07:04,642 INFO  [org.apache.activemq.artemis.core.server] AMQ221007: Server is now live
```

6. Update the `repLiveBroker` configuration to check for a backup server that has failed over, so we don’t have an unaware live broker.

```bash
# For repLiveBroker terminal
vim ${AMQ_HOME}/instances/repLiveBroker/etc/broker.xml
```

```XML
<ha-policy>
 <replication>
    <master>
       <check-for-live-server>true</check-for-live-server>
    </master>
 </replication>
</ha-policy>
```

7. Restart the repLiveBroker

```bash
# Terminal for repLiveBroker
${AMQ_HOME}/instances/repLiveBroker/bin/artemis run
```

>*You will see the live broker start and replicate back from the backup but notice it doesn’t start*

```bash
2024-03-26 03:11:09,781 ERROR [org.apache.activemq.artemis.core.server] AMQ224056: Live server will not fail-back automatically
2024-03-26 03:11:09,909 ERROR [org.apache.activemq.artemis.core.server] AMQ224056: Live server will not fail-back automatically
2024-03-26 03:11:10,301 INFO  [org.apache.activemq.artemis.core.server] AMQ221024: Backup server ActiveMQServerImpl::name=127.0.0.1 is synchronized with live server, nodeID=4741de20-eb3f-11ee-bb4d-525400316e36.
2024-03-26 03:11:14,672 INFO  [org.apache.activemq.artemis.core.server] AMQ221031: backup announced
```

8. Manually terminate the `repBackupBroker` by pressing `CTRL+C` so the `repLiveBroker` takes over.

## Automate the failover

1. Update the `repBackupBroker` configuration.

```bash
# For repBackupBroker terminal
vim ${AMQ_HOME}/instances/repBackupBroker/etc/broker.xml
```

```XML
<ha-policy>
    <replication>
        <slave>
            <allow-failback>true</allow-failback>
        </slave>
    </replication>
</ha-policy>
```

2. Restart the `repBackupBroker` and wait for replication to start

```bash
# Terminal for repLiveBroker
${AMQ_HOME}/instances/repBackupBroker/bin/artemis run
```

3. Terminate the `repLiveBroker` by pressing `CTRL+C`

4. Restart the `repLiveBroker`
   >*You will now see the live broker replicate back then take over as live*

```bash
2024-03-26 03:16:05,188 INFO  [org.apache.activemq.artemis.core.server] AMQ221007: Server is now live
```

End of Lab 10.
