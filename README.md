# OpenShift APIs for Data Protection (OADP) for Red Hat Quay 

Taking a backup of your Quay Registry Database
```
export NAMESPACE=<your Quay Registry namespace>
export STORAGE=<your OADP storage location>

envsubst < backup.yml | oc -n openshift-adp create -f -
``` 

Restoring the backup of your Quay Registry Database

```
oc -n openshift-operators scale --replicas=0 deploy -l operators.coreos.com/quay-operator.openshift-operators
oc -n <quay-registry-namespace> scale --replicas=0 deploy -l quay-component=quay
``` 

After all active connections to the Database have been terminated you can initiate the Restore.

```
envsubst < restore.yml | oc -n openshift-adp create -f -
```

After the restore has finished, you need to restart the Database to ensure all settings are in allignment 
with the `config-secret-bundle`.

```
oc -n <quay-registry-namespace> rollout restart deploy -l quay-component=postgres 
```

After the Database restart you can scale up the Quay Operator which will reconcile the Registry accordingly.

```
oc -n openshift-operators scale --replicas=1 deploy -l operators.coreos.com/quay-operator.openshift-operators
``` 

