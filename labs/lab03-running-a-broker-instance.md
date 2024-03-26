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
./artemis consumer

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
./artemis producer

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

## OCP Running a Broker Instance

### Run a simple broker instance

1. Create the the `ActiveMQArtemis` instance

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
    size: 1
EOF
```

1. Validate the creation of the broker instance and validate the logs

```bash
oc get pods -n amq-broker
NAME          READY   STATUS    RESTARTS   AGE
ex-aao-ss-0   1/1     Running   0          2m7s

oc logs ex-aao-ss-0
2024-03-26 03:34:38,517 INFO  [org.apache.activemq.artemis] AMQ241001: HTTP Server started at http://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:8161
2024-03-26 03:34:38,518 INFO  [org.apache.activemq.artemis] AMQ241002: Artemis Jolokia REST API available at http://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:8161/console/jolokia
2024-03-26 03:34:38,518 INFO  [org.apache.activemq.artemis] AMQ241004: Artemis Console available at http://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:8161/console
```

### Test the broker with the CLI

1. In a new separated terminal, Run the artemis command to consume from the default address.

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
cd ${AMQ_HOME}/instances/mybroker/bin
```

1. Run the artemis command to consume from the default address

```bash
oc exec -ti ex-aao-ss-0 -c ex-aao-container -n amq-broker -- amq-broker/bin/artemis consumer --url tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616 --user admin --password admin
```

1. In another terminal/console do the same but this time to produce messages to the same default address

```bash
oc exec -ti ex-aao-ss-0 -c ex-aao-container -n amq-broker -- amq-broker/bin/artemis producer --url tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616 --user admin --password admin
```

   >  *You will see the producer sending 1000 messages, and the same amount, received by the consumer.*

```bash
#Consumer Output
oc exec -ti ex-aao-ss-0 -c ex-aao-container -n amq-broker -- amq-broker/bin/artemis consumer --url tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616 --user admin --password admin

Connection brokerURL = tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616
Consumer:: filter = null
Consumer ActiveMQQueue[TEST], thread=0 wait until 1000 messages are consumed
Received 1000
Consumer ActiveMQQueue[TEST], thread=0 Consumed: 1000 messages
Consumer ActiveMQQueue[TEST], thread=0 Elapsed time in second : 89 s
Consumer ActiveMQQueue[TEST], thread=0 Elapsed time in milli second : 89621 milli seconds
Consumer ActiveMQQueue[TEST], thread=0 Consumed: 1000 messages
Consumer ActiveMQQueue[TEST], thread=0 Consumer thread finished

#Producer Output
oc exec -ti ex-aao-ss-0 -c ex-aao-container -n amq-broker -- amq-broker/bin/artemis producer --url tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616 --user admin --password admin

Connection brokerURL = tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616
Producer ActiveMQQueue[TEST], thread=0 Started to calculate elapsed time ...

Producer ActiveMQQueue[TEST], thread=0 Produced: 1000 messages
Producer ActiveMQQueue[TEST], thread=0 Elapsed time in second : 4 s
Producer ActiveMQQueue[TEST], thread=0 Elapsed time in milli second : 4157 milli seconds
```

1. Close the producer and consumer terminals

### Management console

1. To access the management web console, get the address of the console route and open it in a browser

```bash
oc get route ex-aao-wconsj-0-svc-rte -n amq-broker -o jsonpath='{"http://"}{.spec.host}{"\n"}'
```

1. Log in to the console using the username/password entered when the AMQ 7 instance was created.

   > In this case is `admin`/`admin`

1. Navigate on console and explore the diferents views.

End of Lab 3.
