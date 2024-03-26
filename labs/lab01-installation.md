# Lab 1. Installation

## RHEL Installation

### Download zip file from Red Hat Customer Portal

1. Login to workstation machine

2. Create the workshop directory

```bash
mkdir -vp $HOME/workshop-amq && cd $HOME/workshop-amq && pwd
```

3. Download the zip file fom AMQ Broker

```bash
wget https://github.com/linuxeroagrio/amq7-broker-workshop/raw/bpd/bin/amq-broker-7.11.0-bin.zip
```

> **NOTE**
> Where does the zip file come from?
> The installation files for AMQ Broker can be downloaded from [Download Red Hat AMQ](https://developers.redhat.com/products/amq/download).


### Install the software by unzipping the file into the designated workshop path ($HOME/workshop-amq)

```bash
unzip amq-broker-7.11.0-bin.zip
cd apache-artemis-2.28.0.redhat-00003
```

> The directory created by extracting the archive will be referenced now on as **AMQ_HOME**

### Directory Structure

```bash
export AMQ_HOME=$HOME/workshop-amq/apache-artemis-2.28.0.redhat-00003

ls -l ${AMQ_HOME}
|-- bin
|-- etc
|-- examples
|-- lib
|-- schema
`-- web
```

| Folder | Description |
| ------ | ----------- |
| bin    | AMQ scripts and native journal lib |
| examples | Full set of runnable JMS and Java EE examples |
| lib    | The AMQ 7 runtime jars and libraries |
| schema | XML Schema to validate the AMQ configuration files |
| web    | Web loaded with hawt.io console, jolokia |

## OCP Installation

### Install AMQ Broker Operator

1. Login to workstation machine

2. Create the subscription for the AMQ Broker Operator
```bash
cat << EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: amq-broker-rhel8
  namespace: openshift-operators
spec:
  channel: 7.11.x
  installPlanApproval: Automatic
  name: amq-broker-rhel8
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

3. Validate the installation of the operator
```bash
oc get sub,ip,csv -n openshift-operators

NAME                                                 PACKAGE            SOURCE             CHANNEL
subscription.operators.coreos.com/amq-broker-rhel8   amq-broker-rhel8   redhat-operators   7.11.x

NAME                                             CSV                                 APPROVAL    APPROVED
installplan.operators.coreos.com/install-vmr5p   amq-broker-operator.v7.11.6-opr-2   Automatic   true

NAME                                                                           DISPLAY                                                   VERSION        REPLACES                            PHASE
clusterserviceversion.operators.coreos.com/amq-broker-operator.v7.11.6-opr-2   Red Hat Integration - AMQ Broker for RHEL 8 (Multiarch)   7.11.6-opr-2   amq-broker-operator.v7.11.5-opr-1   Succeeded
```

End of Lab 1.
