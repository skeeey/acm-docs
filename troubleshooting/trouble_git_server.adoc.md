# Troubleshooting application Git server connection 

Logs from the `open-cluster-management` namespace display failure to clone the Git repository.

## Symptom: Git server connection 

The logs from the subscription controller pod `multicluster-operators-hub-subscription-<random-characters>` in the `open-cluster-management` namespace indicates that it fails to clone the Git repository. You receive a `x509: certificate signed by unknown authority` error, or `BadGateway` error.
 
## Resolving the problem: Git server connection 

**Important:** Upgrade if you are on a previous version.

1. Save [apps.open-cluster-management.io_channels_crd.yaml](https://github.com/stolostron/multicloud-operators-channel/blob/master/deploy/crds/apps.open-cluster-management.io_channels_crd.yaml) as the same file name.
2. On the Red Hat Advanced Cluster Management cluster, run the following command to apply the file: 

+
```
oc apply -f apps.open-cluster-management.io_channels_crd.yaml
```

1. In the `open-cluster-management` namespace, edit the `advanced-cluster-management.<version, example 2.5.0>` CSV, run the following command and edit:

+
```
oc edit csv advanced-cluster-management.<version, example 2.5.0> -n open-cluster-management
```

+
Find the following containers:

+
- `multicluster-operators-standalone-subscription` 
- `multicluster-operators-hub-subscription` 

+
Replace the container images with the container that you want to use:

+
```
quay.io/open-cluster-management/multicluster-operators-subscription:<your image tag>
---- 

+
The update recreates the following pods in the `open-cluster-management` namespace: 


- `multicluster-operators-standalone-subscription-<random-characters>`

- `multicluster-operators-hub-subscription-<random-characters>` 

. Check that the new pods are running with the new docker image. Run the following command, then find the new docker image:
+
```
oc get pod multicluster-operators-standalone-subscription-&lt;random-characters> -n open-cluster-management -o yaml
oc get pod multicluster-operators-hub-subscription-&lt;random-characters> -n open-cluster-management -o yaml
```

. Update the images on managed clusters. 

+
On the hub cluster, run the following command to update the image value in the `multicluster_operators_subscription` key to the image that you want to use:

+
```
oc edit configmap -n open-cluster-management mch-image-manifest-&lt;version, example 2.5.0>
...
data:
multicluster_operators_subscription: &lt;your image with tag>
```

. Restart the existing `multicluster-operators-hub-subscription` pod:

+
```
oc delete pods -n open-cluster-management multicluster-operators-hub-subscription--&lt;random-characters>
```

+
This recreates the `application-manager-<random-characters>` pod in `open-cluster-management-agent-addon` namespace on the managed cluster. 

. Check that the new pod is running with the new docker image.

. When you create an application through the console or the CLI, add `insecureSkipVerify: true' in the channel spec manually. See the following example:

+
[source,yaml]
```
apiVersion: apps.open-cluster-management.io/v1
kind: Channel
metadata:
labels:
  name: sample-channel
  namespace: sample
spec:
  type: GitHub
  pathname: &lt;Git URL>
  insecureSkipVerify: true
```
