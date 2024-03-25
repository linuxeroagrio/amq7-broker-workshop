# Lab 1. Installation

## RHEL Installation

### Download zip file from Red Hat Customer Portal

1. Login to workstation machine

1. Create the workshop directory

```bash
mkdir -vp $HOME/workshop-amq && cd $HOME/workshop-amq && pwd
```

3. Download the zip file fom AMQ Broker

```bash
wget https://developers.redhat.com/content-gateway/file/amq/GA_April_2023/amq-broker-7.11.0-bin.zip
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

1. 
End of Lab 1.
