# Lab 6. Securing a Queue

## RHEL Securing a Queue

### Adding security to a Queue

1. In the workstation machine, add a new user `myuser` to the broker instance

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
echo "myuser=mypassword" >> ${AMQ_HOME}/instances/mybroker/etc/artemis-users.properties
echo "mygroup=myuser" >> ${AMQ_HOME}/instances/mybroker/etc/artemis-roles.properties
```

2. Update the security settings to only allow **send** to the new user’s role `mygroup` in the `${AMQ_HOME}/etc/broker.xml` file

```bash
vim ${AMQ_HOME}/instances/mybroker/etc/broker.xml
```

```XML
  <security-settings>
      <security-setting match="#">
     ...
         <permission type="consume" roles="admin"/>
         <permission type="browse" roles="admin"/>
         <permission type="send" roles="admin,mygroup"/>
         <!-- we need this otherwise ./artemis data imp wouldn't work -->
         <permission type="manage" roles="admin"/>
      </security-setting>
  </security-settings>
```

3. Start the broker or restart in case it’s already running.

```bash
${AMQ_HOME}/instances/mybroker/bin/artemis-service restart
Restarting artemis-service
artemis-service is now running (4723)
```

### Testing the Security Settings

1. In a new terminal, test sending messages to the previously created topic testTopic

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
${AMQ_HOME}/instances/mybroker/bin/artemis producer --destination topic://testTopic --message-count 10 --user myuser --password mypassword
```

2. In a third terminal, test consuming messages from the same topic using the new user.

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003
${AMQ_HOME}/instances/mybroker/bin/artemis consumer --destination topic://testTopic --message-count 10 --user myuser --password mypassword
```

You will get an ActiveMQSecurityException

```bash
Consumer ActiveMQTopic[testTopic], thread=0 wait until 10 messages are consumed
javax.jms.JMSSecurityException: AMQ229213: User: myuser does not have permission='CREATE_NON_DURABLE_QUEUE' for queue e6595d13-6ad2-4e5a-82b2-fc6bc34b0dbf on address testTopic
	at org.apache.activemq.artemis.core.protocol.core.impl.ChannelImpl.sendBlocking(ChannelImpl.java:558)
	at org.apache.activemq.artemis.core.protocol.core.impl.ChannelImpl.sendBlocking(ChannelImpl.java:450)
	at org.apache.activemq.artemis.core.protocol.core.impl.ActiveMQSessionContext.createQueue(ActiveMQSessionContext.java:854)
	at org.apache.activemq.artemis.core.client.impl.ClientSessionImpl.internalCreateQueue(ClientSessionImpl.java:2071)
	at org.apache.activemq.artemis.core.client.impl.ClientSessionImpl.createQueue(ClientSessionImpl.java:316)
	at org.apache.activemq.artemis.jms.client.ActiveMQSession.createTemporaryQueue(ActiveMQSession.java:1262)
	at org.apache.activemq.artemis.jms.client.ActiveMQSession.createConsumer(ActiveMQSession.java:842)
	at org.apache.activemq.artemis.jms.client.ActiveMQSession.createConsumer(ActiveMQSession.java:477)
	at org.apache.activemq.artemis.jms.client.ActiveMQSession.createConsumer(ActiveMQSession.java:449)
	at org.apache.activemq.artemis.cli.commands.messages.ConsumerThread.consume(ConsumerThread.java:178)
	at org.apache.activemq.artemis.cli.commands.messages.ConsumerThread.run(ConsumerThread.java:67)
Caused by: ActiveMQSecurityException[errorType=SECURITY_EXCEPTION message=AMQ229213: User: myuser does not have permission='CREATE_NON_DURABLE_QUEUE' for queue e6595d13-6ad2-4e5a-82b2-fc6bc34b0dbf on address testTopic]
	... 11 more
Consumer ActiveMQTopic[testTopic], thread=0 Consumer thread finished
```

3. Exit from the consumer and producer terminal sessions.


## OCP Securing a Queue

### Adding security to a Queue

1. In the workstation machine, add a `ActiveMQArtemisSecurity` resource

```bash
cat <<EOF | oc apply -f -
apiVersion: broker.amq.io/v1alpha1
kind: ActiveMQArtemisSecurity
metadata:
  name: ex-prop
  namespace: amq-broker
spec:
  loginModules:
    propertiesLoginModules:
      - name: "prop-module"
        users:
          - name: "myuser"
            password: "mypassword"
            roles:
              - "mygroup"
  securityDomains:
    brokerDomain:
      name: "activemq"
      loginModules:
        - name: "prop-module"
          flag: "sufficient"
  securitySettings:
    broker:
      - match: "#"
        permissions:
          - operationType: "send"
            roles:
              - "mygroup"
EOF

oc get ActiveMQArtemisSecurity ex-prop -n amq-broker

NAME      AGE
ex-prop   22s
```

### Testing the Security Settings

1. In a new terminal, test sending messages to the previously created topic testTopic

```bash
oc exec -ti ex-aao-ss-0 -c ex-aao-container -n amq-broker -- amq-broker/bin/artemis producer --url tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616 --user myuser --password mypassword --destination topic://testTopic --message-count 10
```

2. In a third terminal, test consuming messages from the same topic using the new user.

```bash
oc exec -ti ex-aao-ss-0 -c ex-aao-container -n amq-broker -- amq-broker/bin/artemis consumer --url tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616 --user myuser --password mypassword --destination topic://testTopic --message-count 10
```

You will get an ActiveMQSecurityException

```bash
oc exec -ti ex-aao-ss-0 -c ex-aao-container -n amq-broker -- amq-broker/bin/artemis consumer --url tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616 --user myuser --password mypassword --destination topic://testTopic --message-count 10

Connection brokerURL = tcp://ex-aao-ss-0.ex-aao-hdls-svc.amq-broker.svc.cluster.local:61616
Consumer:: filter = null
Consumer ActiveMQTopic[testTopic], thread=0 wait until 10 messages are consumed
javax.jms.JMSSecurityException: AMQ229213: User: myuser does not have permission='CREATE_NON_DURABLE_QUEUE' for queue 66d29a9f-7063-42b0-9828-ec069bf77f74 on address testTopic
	at org.apache.activemq.artemis.core.protocol.core.impl.ChannelImpl.sendBlocking(ChannelImpl.java:560)
	at org.apache.activemq.artemis.core.protocol.core.impl.ChannelImpl.sendBlocking(ChannelImpl.java:452)
	at org.apache.activemq.artemis.core.protocol.core.impl.ActiveMQSessionContext.createQueue(ActiveMQSessionContext.java:859)
	at org.apache.activemq.artemis.core.client.impl.ClientSessionImpl.internalCreateQueue(ClientSessionImpl.java:2071)
	at org.apache.activemq.artemis.core.client.impl.ClientSessionImpl.createQueue(ClientSessionImpl.java:316)
	at org.apache.activemq.artemis.jms.client.ActiveMQSession.createTemporaryQueue(ActiveMQSession.java:1287)
	at org.apache.activemq.artemis.jms.client.ActiveMQSession.createConsumer(ActiveMQSession.java:848)
	at org.apache.activemq.artemis.jms.client.ActiveMQSession.createConsumer(ActiveMQSession.java:481)
	at org.apache.activemq.artemis.jms.client.ActiveMQSession.createConsumer(ActiveMQSession.java:453)
	at org.apache.activemq.artemis.cli.commands.messages.ConsumerThread.consume(ConsumerThread.java:178)
	at org.apache.activemq.artemis.cli.commands.messages.ConsumerThread.run(ConsumerThread.java:67)
Caused by: ActiveMQSecurityException[errorType=SECURITY_EXCEPTION message=AMQ229213: User: myuser does not have permission='CREATE_NON_DURABLE_QUEUE' for queue 66d29a9f-7063-42b0-9828-ec069bf77f74 on address testTopic]
	... 11 more
Consumer ActiveMQTopic[testTopic], thread=0 Consumer thread finished
```

End of Lab 6.
