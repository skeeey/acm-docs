# Known issues for networking

Review the known issues for Submariner. The following list contains known issues for this release, or known issues that continued from the previous release. 

For your Red Hat OpenShift Container Platform cluster, see [OpenShift Container Platform known issues](https://docs.redhat.com/documentation/en-us/openshift_container_platform/4.15/html/release_notes/#ocp-4-15-known-issues). 

For more about deprecations and removals, see [Deprecations and removals for Red Hat Advanced Cluster Management](../release_notes/acm_deprecate_remove.adoc#deprecations-removals-acm).

## Submariner known issues

See the following known issues and limitations that might occur while using networking features.

### Without _ClusterManagementAddon_ submariner add-on fails

For versions 2.8 and earlier, when you install Red Hat Advanced Cluster Management, you also deploy the `submariner-addon` component with the Operator Lifecycle Manager. If you did not create a `MultiClusterHub` custom resource, the `submariner-addon` pod sends an error and prevents the operator from installing. 

The following notification occurs because the `ClusterManagementAddon` custom resource definition is missing:

```
graceful termination failed, controllers failed with error: the server could not find the requested resource (post clustermanagementaddons.addon.open-cluster-management.io)
```

The `ClusterManagementAddon` resource is created by the `cluster-manager` deployment, however, this deployment becomes available when the `MultiClusterEngine` components are installed on the cluster. 

If there is not a `MultiClusterEngine` resource that is already available on the cluster when the `MultiClusterHub` custom resource is created,  the `MultiClusterHub` operator deploys the `MultiClusterEngine` instance and the operator that is required, which resolves the previous error.

### Submariner add-on resources not cleaned up properly when managed clusters are imported 

If the `submariner-addon` component is set to `false` within `MultiClusterHub` (MCH) operator, then the `submariner-addon` finalizers are not cleaned up properly for the managed cluster resources. Since the finalizers are not cleaned up properly, this prevents the `submariner-addon` component from being disabled within the hub cluster. 

### Submariner install plan limitation

The Submariner install plan does not follow the overall install plan settings. Therefore, the operator management screen cannot control the Submariner install plan. By default, Submariner install plans are applied automatically, and the Submariner addon is always updated to the latest available version corresponding to the installed  Red Hat Advanced Cluster Management version. To change this behavior, you must use a customized Submariner subscription. 

### Limited headless services support

Service discovery is not supported for headless services without selectors when using Globalnet.

### Deployments that use VXLAN when NAT is enabled are not supported

Only non-NAT deployments support Submariner deployments with the VXLAN cable driver.

### OVN Kubernetes requires OCP 4.11 and later

If you are using the OVN Kubernetes CNI network, you need Red Hat OpenShift 4.11 or later.

### Self-signed certificates might prevent connection to broker

Self-signed certificates on the broker might prevent joined clusters from connecting to the broker. The connection fails with certificate validation errors. You can disable broker certificate validation by setting `InsecureBrokerConnection` to `true` in the relevant `SubmarinerConfig` object. See the following example:

```yaml
apiVersion: submarineraddon.open-cluster-management.io/v1alpha1
kind: SubmarinerConfig
metadata:
   name: submariner
   namespace: <managed-cluster-namespace>
spec:
   insecureBrokerConnection: true
```

### Submariner only supports OpenShift SDN or OVN Kubernetes

Submariner only supports Red Hat OpenShift Container Platform clusters that use the OpenShift SDN or the OVN-Kubernetes Container Network Interface (CNI) network provider.

### Command limitation on Microsoft Azure clusters

The `subctl diagnose firewall inter-cluster` command does not work on Microsoft Azure clusters.

### Automatic upgrade not working with custom _CatalogSource_ or _Subscription_

Submariner is automatically upgraded when Red Hat Advanced Cluster Management for Kubernetes is upgraded. The automatic upgrade might fail if you are using a custom `CatalogSource` or `Subscription`.

To make sure automatic upgrades work when installing Submariner on managed clusters, you must set the `spec.subscriptionConfig.channel` field to `stable-0.15` in the `SubmarinerConfig` custom resource for each managed cluster.

### Submariner conflicts with IPsec-enabled OVN-Kubernetes deployments

IPsec tunnels that are created by IPsec-enabled OVN-Kubernetes deployments might conflict with IPsec tunnels that are created by Submariner. Do not use OVN-Kubernetes in IPsec mode with Submariner.

### Uninstall Submariner before removing _ManagedCluster_ from a _ManageClusterSet_

If you remove a cluster from a `ClusterSet`, or move a cluster to a different `ClusterSet`, the Submariner installation is no longer valid.

You must uninstall Submariner before moving or removing a `ManagedCluster` from a `ManageClusterSet`. If you don’t uninstall Submariner, you cannot uninstall or reinstall Submariner anymore and Submariner stops working on your `ManagedCluster`.
