# OpenShift-OFD-guide

## Installation
1. Install the two following operators in the UI.
``` Local Storage ```
&
``` OpenShift Data Foundation ```

### Installation using CLI
#### Label all worker nodes
```
oc get nodes -l node-role.kubernetes.io/worker=
NAME       STATUS   ROLES    AGE   VERSION
worker01   Ready    worker   14d   v1.20.0+a0b09eb
worker02   Ready    worker   14d   v1.20.0+a0b09eb
worker03   Ready    worker   14d   v1.20.0+a0b09eb
```
Given the name of your notes are "worker01" etc we label them like this
```
oc label nodes -l node-role.kubernetes.io/worker= cluster.ocs.openshift.io/openshift-storage=
node/worker01 labeled
node/worker02 labeled
node/worker03 labeled
```
We now analyze the disks available on one of the workernotes
```
oc debug node/worker01 -- lsblk --paths --nodeps
```
Based on what we see here we want to go for space on /vdb and /vdc given they are not in use and have space available 

Now we create the "openshift-local-storage" namespace (more info in README_LocalStorage.yaml) and change into it.
```
oc adm new-project openshift-local-storage

oc project openshift-local-storage
```

Now we create an "OperatorGroup" which defines the scope of the operators that will be deployed in our namespace. Modify and use the "lso-operatorgroup.yaml" for this.
- This is done to limit the scope of the operator to our namespace for security or resource management reasons.
```
oc apply -f lso-operatorgroup.yaml
```

