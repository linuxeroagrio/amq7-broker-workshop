# Environment

## Lab environment

The lab environment provides a virtualized setup for developing and running Kafka applications on OpenShift.
It consists of:

* an OpenShift cluster with three control plane nodes and three worker nodes;
* a workstation machine;

During the lab you will interact with the OpenShift cluster via CLI from the workstation and with the OpenShift web console from the browser available on your station.
Make note of the URLs that you will use during the lab.

|===
| Machine | URL | Access
| Workstation | `ssh.ocpv01.<WORKSTATION_HOST_ID>.infra.demo.redhat.com` | SSH (custom port)
| OpenShift console | `console-openshift-console.apps.cluster-<GUID>.dynamic.redhatworkshops.io` | HTTPS (port 443)
| OpenShift API | `api.cluster-<GUID>.dynamic.redhatworkshops.io` | HTTPS (port 6443)
|===

Every lab assumes that you have access to the workstation via SSH for CLI interaction and to the OpenShift web console.

Before you start any lab, make sure you are logged into the workstation via SSH with the password and port that has been provided by the instructors.

     ssh lab-user@ssh.ocpv01.<WORKSTATION_HOST_ID>.infra.demo.redhat.com -p <CUSTOM-SSH-PORT>

By default, you have access to the cluster API via workstation machine login with system:admin user, but, if you prefer, you can access with an `admin` user.
It should be used for logging in both via CLI and web console.
The password will be provided by the instructors.

[NOTE]
.Remember your GUID
At the start of the lab you have been provided with a GUID, which uniquely identifies your lab environment.
In the following instructions, replace the `<GUID>` placeholder with your GUID.
For example, if your GUID is `a2b3`, the URL `console-openshift-console.apps.cluster-<GUID>.dynamic.redhatworkshops.io` becomes `console-openshift-console.apps.cluster-a2b3.dynamic.redhatworkshops.io`.

Now, let's start the lab.