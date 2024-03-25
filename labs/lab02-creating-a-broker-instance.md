# Lab 2. Creating a Broker Instance

## RHEL Broker Instance

The `artemis` script located in the `bin` folder is the starting point to manage our AMQ installation.

### Use the script to create a new broker instance

1. Ensure that the JDK is installed on the workstation

```bash
sudo yum -y install java-11-openjdk java-11-openjdk-devel java-17-openjdk-headless
```

2. Execute the following commands to create a new broker in the instances folder.

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003

${AMQ_HOME}/bin/artemis create ${AMQ_HOME}/instances/mybroker
```

3. The command will ask a series of questions in order to configure the broker instance:

   * Use `admin` as username and `admin` as password for the mandatory user
   * Add the `admin` role to the admin user
   * Allow anonymous access by typing `Y` when asked

   > As the AMQ Broker requires to persist messages, we need to have enough free space in the disk to store the messages, please chekc that there is at least 10% available.

```bash
done! Your system can make 0.18 writes per millisecond, your journal-buffer-timeout will be 5624000

You can now start the broker by executing:  

   "/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/mybroker/bin/artemis" run

Or you can run the broker in the background using:

   "/home/lab-user/workshop-amq/apache-artemis-2.28.0.redhat-00003/instances/mybroker/bin/artemis-service" start
```

### Directory Structure

```bash
ls -l ${AMQ_HOME}/instances/mybroker
|-- bin
|-- data
|-- etc
|-- lib
|-- log
`-- tmp
```

| Folder | Description |
| ------ | ----------- |
| bin    | AMQ instance command line scripts to launch and manage |
| etc    | Broker instance configuration files |
| lib    | Extra libraries                                        |
| log    | Instance log files |
| data   | Journal files storage for persistent messages          |
| tmp | Temporary files |

## OCP Broker Instance

1. Create the `amq-broker` project

```bash
oc new-project amq-broker
```

2. Create the the `ActiveMQArtemis` instance

```bash
cat << EOF | oc apply -f -
apiVersion: broker.amq.io/v1beta1
kind: ActiveMQArtemis
metadata:
  name: ex-aao
  application: ex-aao-app
  namespace: amq-broker
spec:
  adminUser: admin
  adminPassword: admin
  deploymentPlan:
    image: placeholder
    jolokiaAgentEnabled: false
    journalType: nio
    managementRBACEnabled: true
    messageMigration: false
    persistenceEnabled: false
    requireLogin: true
    size: 0
EOF
```

3. Validate if the instance had a `Valid` value

```bash
oc get ActiveMQArtemis ex-aao -n amq-broker -o jsonpath='{.status.conditions[0].type} {.status.conditions[0].status}{"\n"}'

Valid True
```

End of Lab 2.
