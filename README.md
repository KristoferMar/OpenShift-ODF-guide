# OpenShift-OFD-guide

## Installation
1. Install the two following operators in the UI.
``` Local Storage ```
&
``` OpenShift Data Foundation ```

### Installation using CLI
#### Label all worker nodes
```
[kristofer@workstation ~]$ oc get nodes -l node-role.kubernetes.io/worker=
NAME       STATUS   ROLES    AGE   VERSION
worker01   Ready    worker   14d   v1.20.0+a0b09eb
worker02   Ready    worker   14d   v1.20.0+a0b09eb
worker03   Ready    worker   14d   v1.20.0+a0b09eb
```
Given the name of your notes are "worker01" etc we label them like this
```
[kristofer@workstation ~]$ oc label nodes \
  -l node-role.kubernetes.io/worker= \
  cluster.ocs.openshift.io/openshift-storage=
node/worker01 labeled
node/worker02 labeled
node/worker03 labeled
```
