# Must-gather for a hosted cluster

If you experience issues with hosted control plane clusters, you can run the `must-gather` command to gather information to help you with troubleshooting.

## About the must-gather command for hosted clusters

The command generates output for the managed cluster and the hosted cluster.

* Data from the multicluster engine operator hub cluster:
  * Cluster-scoped resources: These resources are node definitions of the management cluster.
  * The `hypershift-dump` compressed file: This file is useful if you need to share the content with other people.
  * Namespaced resources: These resources include all of the objects from the relevant namespaces, such as config maps, services, events, and logs.
  * Network logs: These logs include the OVN northbound and southbound databases and the status for each one.
  * Hosted clusters: This level of output involves all of the resources inside of the hosted cluster.
* Data from the hosted cluster:
  * Cluster-scoped resources: These resources include all of the cluster-wide objects, such as nodes and CRDs.
  * Namespaced resources: These resources include all of the objects from the relevant namespaces, such as config maps, services, events, and logs.

Although the output does not contain any secret objects from the cluster, it can contain references to the names of the secrets.

## Prerequisites

To gather information by running the must-gather command, you must meet the following prerequisites:

* You must ensure that the `kubeconfig` file is loaded and is pointing to the multicluster engine operator hub cluster.
* You must have cluster-admin access to the multicluster engine operator hub cluster.
* You must have the name value for the `HostedCluster` resource and the namespace where the custom resource is deployed.

## Entering the must-gather command for hosted clusters

1. Enter the following command to collect information about the hosted cluster. In the command, the `hosted-cluster-namespace=HOSTEDCLUSTERNAMESPACE` parameter is optional; if you do not include it, the command runs as though the hosted cluster is in the default namespace, which is `clusters`.

+
```
oc adm must-gather --image=quay.io/stolostron/backplane-must-gather:SNAPSHOTNAME /usr/bin/gather hosted-cluster-namespace=HOSTEDCLUSTERNAMESPACE hosted-cluster-name=HOSTEDCLUSTERNAME
```

1. To save the results of the command to a compressed file, include the `--dest-dir=NAME` parameter, replacing `NAME` with the name of the directory where you want to save the results:

+
```
oc adm must-gather --image=quay.io/stolostron/backplane-must-gather:SNAPSHOTNAME /usr/bin/gather hosted-cluster-namespace=HOSTEDCLUSTERNAMESPACE hosted-cluster-name=HOSTEDCLUSTERNAME --dest-dir=NAME ; tar -cvzf NAME.tgz NAME
```

## Entering the must-gather command in a disconnected environment

Complete the following steps to run the `must-gather` command in a disconnected environment:

1. In a disconnected environment, mirror the Red Hat operator catalog images into their mirror registry. For more information, see [Install in disconnected network environments](../install/install_disconnected.adoc#install-on-disconnected-networks).
2. Run the following command to extract logs, which reference the image from their mirror registry:

+
```
REGISTRY=registry.example.com:5000
IMAGE=$REGISTRY/multicluster-engine/must-gather-rhel8@sha256:ff9f37eb400dc1f7d07a9b6f2da9064992934b69847d17f59e385783c071b9d8

oc adm must-gather --image=$IMAGE /usr/bin/gather hosted-cluster-namespace=HOSTEDCLUSTERNAMESPACE hosted-cluster-name=HOSTEDCLUSTERNAME --dest-dir=./data
```

## Additional resources

* For more information about troubleshooting hosted control planes, see [Troubleshooting hosted control planes](https://docs.redhat.com/documentation/en-us/openshift_container_platform/4.14/html/hosted_control_planes/troubleshooting-hosted-control-planes) in the OpenShift Container Platform documentation.
