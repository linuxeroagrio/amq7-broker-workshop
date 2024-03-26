# Lab 3. Running a Broker Instance

## RHEL Running a Broker Instance

### Run the broker in interactive mode

1. Open a new terminal and login to the workstation machine with the following command

   ```bash
   ssh lab-user@ssh.ocpv01.<WORKSTATION_HOST_ID>.infra.demo.redhat.com -p <CUSTOM-SSH-PORT> -L 8161:127.0.0.1:8161
   ```
   > **Note**
   > Use your own workstation host id and ssh ports provided by instructor.

1. Once the broker is created, navigate to the `mybroker` instance `bin` folder if not already there

   ```bash
   export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
   cd ${AMQ_HOME}/instances/mybroker/bin
   ```

1. Run the `artemis` script to start the broker in interactive mode

   ```bash
   ./artemis run
   ```

The broker will produce output similar to the following:

```bash
INFO  [org.apache.activemq.artemis] AMQ241001: HTTP Server started at http://localhost:8161
INFO  [org.apache.activemq.artemis] AMQ241002: Artemis Jolokia REST API available at http://localhost:8161/console/jolokia
INFO  [org.apache.activemq.artemis] AMQ241004: Artemis Console available at http://localhost:8161/console
```

### Test the broker with the CLI

1. In a new separated terminal, navigate to the bin folder of the broker instance.

   ```bash
   export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
   cd ${AMQ_HOME}/instances/mybroker/bin
   ```

1. Run the artemis command to consume from the default address

   ```bash
   ./artemis consumer
   ```

1. In another terminal/console do the same but this time to produce messages to the same default address

   ```bash
   export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
   cd ${AMQ_HOME}/instances/mybroker/bin
   ./artemis producer
   ```

   >  *You will see the producer sending 1000 messages, and the same amount, received by the consumer.*

```bash
#Consumer Output
[lab-user@bastion bin]$ ./artemis consumer
Connection brokerURL = tcp://localhost:61616
Consumer:: filter = null
Consumer ActiveMQQueue[TEST], thread=0 wait until 1000 messages are consumed
Received 1000
Consumer ActiveMQQueue[TEST], thread=0 Consumed: 1000 messages
Consumer ActiveMQQueue[TEST], thread=0 Elapsed time in second : 191 s
Consumer ActiveMQQueue[TEST], thread=0 Elapsed time in milli second : 191408 milli seconds
Consumer ActiveMQQueue[TEST], thread=0 Consumed: 1000 messages
Consumer ActiveMQQueue[TEST], thread=0 Consumer thread finished

#Producer Output
[lab-user@bastion bin]$ ./artemis producer
Connection brokerURL = tcp://localhost:61616
Producer ActiveMQQueue[TEST], thread=0 Started to calculate elapsed time ...

Producer ActiveMQQueue[TEST], thread=0 Produced: 1000 messages
Producer ActiveMQQueue[TEST], thread=0 Elapsed time in second : 5 s
Producer ActiveMQQueue[TEST], thread=0 Elapsed time in milli second : 5947 milli seconds
```

1. Close the producer and consumer terminals

### Management console

1. To access the management web console, open a browser and goto http://localhost:8161
1. Log in to the console using the username/password entered when the AMQ 7 instance was created.

   > In this case is `admin`/`admin`

1. Navigate on console and explore the diferents views.

### Running as a Service

1. Shutdown the running instance by pressing `CTRL+C`.

1. Start the instance in service mode

```bash
./artemis-service start
Starting artemis-service
artemis-service is now running (2656)
```

1. Validate the process id.

```bash
ps -fea | grep -i artemis

lab-user    2656       1 16 23:24 pts/0    00:00:07 java -XX:AutoBoxCacheMax=20000 -XX:+PrintClassHistogram -XX:+UseG1GC -XX:+UseStringDeduplication -Xms512M -Xmx2G -Dhawtio.disableProxy=true -Dhawtio.realm=activemq -Dhawtio.offline=true -Dhawtio.rolePrincipalClasses=org.apache.activemq.artemis.spi.core.security.jaas.RolePrincipal -Dhawtio.http.strictTransportSecurity=max-age=31536000;includeSubDomains;preload -Djolokia.policyLocation=file:/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/mybroker/etc/jolokia-access.xml -Dhawtio.role=amq -Djava.security.auth.login.config=/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/mybroker/etc/login.config -classpath /home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/lib/artemis-boot.jar -Dartemis.home=/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003 -Dartemis.instance=/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/mybroker -Djava.library.path=/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/bin/lib/linux-x86_64 -Djava.io.tmpdir=/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/mybroker/tmp -Ddata.dir=/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/mybroker/data -Dartemis.instance.etc=/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/mybroker/etc org.apache.activemq.artemis.boot.Artemis run
```

1. Stop the instace and start it again

```bash
./artemis-service stop
Gracefully Stopping artemis-service
```

```bash
./artemis-service start
Starting artemis-service
artemis-service is now running (2656)
```

End of Lab 3.
