# Setting up OpenShift clusters

## Installing ODF operator on the managed clusters

Install the OpenShift Data Foundation operator accepting the defaults
When the installation ends you must refresh the web page to load the UI
for creating a new storage system.

> [!IMPORTANT]
> Do not create the storage system from the UI! (reason not clear)

## Preparing the nodes

The odf worker nodes must have the label `cluster.ocs.openshift.io/openshift-storage: ''`.

Label the worker nodes:

```
$ oc label nodes --selector node-role.kubernetes.io/worker= \
    cluster.ocs.openshift.io/openshift-storage= --context perf2
node/perf2-t4wdx-odf-0-bdq24 labeled
node/perf2-t4wdx-odf-0-hcxlr labeled
node/perf2-t4wdx-odf-0-hpjz2 labeled

$ oc label nodes --selector node-role.kubernetes.io/worker= \
    cluster.ocs.openshift.io/openshift-storage= --context perf3
node/perf3-fc4vx-odf-0-l4xf9 labeled
node/perf3-fc4vx-odf-0-mg8cq labeled
node/perf3-fc4vx-odf-0-x96j5 labeled
```

## Prepare the openshift-storage namespace

Label namespace openshift-storage for monitoring

```
$ oc label namespace openshift-storage openshift.io/cluster-monitoring=true --context perf2
namespace/openshift-storage labeled

$ oc label namespace openshift-storage openshift.io/cluster-monitoring=true --context perf3
namespace/openshift-storage labeled
```

## Create a storage cluster

Create a storage cluster on the managed clusters:

```
$ oc create -f cluster/storagecluster.yaml -n openshift-storage --context perf2
storagecluster.ocs.openshift.io/ocs-storagecluster created

$ oc create -f cluster/storagecluster.yaml -n openshift-storage --context perf3
storagecluster.ocs.openshift.io/ocs-storagecluster created
```

Wait until the storage cluster is ready. This can take few minutes.

```
$ oc wait storagecluster ocs-storagecluster -n openshift-storage \
    --for=jsonpath='{.status.phase}=Ready' --timeout 5m --context perf2
storagecluster.ocs.openshift.io/ocs-storagecluster condition met

$ oc wait storagecluster ocs-storagecluster -n openshift-storage \
    --for=jsonpath='{.status.phase}=Ready' --timeout 5m --context perf3
storagecluster.ocs.openshift.io/ocs-storagecluster condition met
```

## Installing gitops on the hub

For creating application sets base applications, we need to install and
configure the OpenShift GitOps operator on the hub.

> [!NOTE]
> This allows only push mode. For pull mode we need more instructions.

Install the Red Hat OpenShift GitOps operator on the hub accepting defaults.

Register the managed clusters to gitops:

```
$ oc apply -k gitops/ --context perf1
gitopscluster.apps.open-cluster-management.io/dr-gitops created
placement.cluster.open-cluster-management.io/all-openshift-clusters created
managedclustersetbinding.cluster.open-cluster-management.io/dr created
```

To verify gitops deployment check the Cluster Argo CD console at:
https://{cluster-url}/settings/clusters

For more info see
[Red Hat OpenShift GitOps release notes](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.12/html/cicd/gitops).
