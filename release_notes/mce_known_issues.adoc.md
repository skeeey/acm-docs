# Known issues and limitations for Cluster lifecycle with multicluster engine

Review the known issues and limitations for _Cluster lifecycle with multicluster engine operator_ for this release, or known issues that continued from the previous release. 

Cluster management known issues and limitations are part of the _Cluster lifecycle with multicluster engine operator_ documentation. Known issues for multicluster engine integrated _with_ Red Hat Advanced Cluster Management are documented in the [Release notes for Red Hat Advanced Cluster Management](../../release_notes/acm_release_notes.adoc#acm-release-notes). 

**Important:** OpenShift Container Platform release notes are not documented in this product documentation. For your OpenShift Container Platform cluster, see [OpenShift Container Platform release notes](https://docs.redhat.com/documentation/en-us/openshift_container_platform/4.15).

* [Installation](#installation-)
* [Cluster management](#cluster)
* [Central infrastructure management](#central-infrastructure-management)

## Installation 

Learn about known issues and limitations during multicluster engine operator installation.

### Status stuck when installing on OpenShift Service on AWS with hosted control plane cluster

Installation status might get stuck in the `Installing` state when you install multicluster engine operator on a OpenShift Service on AWS with hosted control planes cluster. The `local-cluster` might also remain in the `Unknown` state. 

When you check the `klusterlet-agent` pod log in the `open-cluster-management-agent` namespace on your hub cluster, you see an error that resembles the following:

```terminal
E0809 18:45:29.450874       1 reflector.go:147] k8s.io/client-go@v0.29.4/tools/cache/reflector.go:229: Failed to watch *v1.CertificateSigningRequest: failed to list *v1.CertificateSigningRequest: Get "https://api.xxx.openshiftapps.com:443/apis/certificates.k8s.io/v1/certificatesigningrequests?limit=500&resourceVersion=0": tls: failed to verify certificate: x509: certificate signed by unknown authority
```

To resolve the problem, configure the hub cluster API server verification strategy. Complete the following steps:

1. Create a `KlusterletConfig` resource with name `global` if it does not exist.
2. Set the `spec.hubKubeAPIServerConfig.serverVerificationStrategy` to `UseSystemTruststore`. See the following example:

+
```yaml
apiVersion: config.open-cluster-management.io/v1alpha1
kind: KlusterletConfig
metadata:
  name: global
spec:
  hubKubeAPIServerConfig:
    serverVerificationStrategy: UseSystemTruststore
```

1. Apply the resource by running the following command on the hub cluster. Replace `<filename>` with the name of your file:

+
```bash
oc apply -f <filename>
```

1. If the `local-cluster` state does not recover in one minute, export and decode the `import.yaml` file by running the following command on the hub cluster:

+
```bash
oc get secret local-cluster-import -n local-cluster -o jsonpath={.data.import\.yaml} | base64 --decode > import.yaml
```

1. Apply the file by running the following command on the hub cluster:

+
```bash
oc apply -f import.yaml
```

### _installNamespace_ field can only have one value

When enabling the `managed-serviceaccount` add-on, the `installNamespace` field in the `ManagedClusterAddOn` resource must have `open-cluster-management-agent-addon` as the value. Other values are ignored. The `managed-serviceaccount` add-on agent is always deployed in the `open-cluster-management-agent-addon` namespace on the managed cluster.

## Cluster

Learn about known issues and limitations for Cluster lifecycle with multicluster engine operator, such as issues with creating, discovering, importing, and removing clusters, and more cluster management issues for multicluster engine operator. 

### Limitation with _nmstate_

Develop quicker by configuring copy and paste features. To configure the `copy-from-mac` feature in the `assisted-installer`, you must add the `mac-address` to the `nmstate` definition interface and the `mac-mapping` interface. The `mac-mapping` interface is provided outside the `nmstate` definition interface. As a result, you must provide the same `mac-address` twice. 

If you have a different version of VolSync installed, replace `v0.6.0` with your installed version. 

### Deleting a managed cluster set does not automatically remove its label

After you delete a `ManagedClusterSet`, the label that is added to each managed cluster that associates the cluster to the cluster set is not automatically removed. Manually remove the label from each of the managed clusters that were included in the deleted managed cluster set. The label resembles the following example: `cluster.open-cluster-management.io/clusterset:<ManagedClusterSet Name>`.

### ClusterClaim error

If you create a Hive `ClusterClaim` against a `ClusterPool` and manually set the `ClusterClaimspec` lifetime field to an invalid golang time value, the product stops fulfilling and reconciling all `ClusterClaims`, not just the malformed claim.  

You see the following error in the `clusterclaim-controller` pod logs, which is a specific example with the `PoolName` and invalid `lifetime` included:

```
E0203 07:10:38.266841       1 reflector.go:138] sigs.k8s.io/controller-runtime/pkg/cache/internal/informers_map.go:224: Failed to watch *v1.ClusterClaim: failed to list *v1.ClusterClaim: v1.ClusterClaimList.Items: []v1.ClusterClaim: v1.ClusterClaim.v1.ClusterClaim.Spec: v1.ClusterClaimSpec.Lifetime: unmarshalerDecoder: time: unknown unit "w" in duration "1w", error found in #10 byte of ...|time":"1w"}},{"apiVe|..., bigger context ...|clusterPoolName":"policy-aas-hubs","lifetime":"1w"}},{"apiVersion":"hive.openshift.io/v1","kind":"Cl|...
```

You can delete the invalid claim.

If the malformed claim is deleted, claims begin successfully reconciling again without any further interaction.

### The product channel out of sync with provisioned cluster

The `clusterimageset` is in `fast` channel, but the provisioned cluster is in `stable` channel. Currently the product does not sync the `channel` to the provisioned OpenShift Container Platform cluster. 

Change to the right channel in the OpenShift Container Platform console. Click ***Administration*** > ***Cluster Settings*** > ***Details Channel***.

### Selecting a subnet is required when creating an on-premises cluster

When you create an on-premises cluster using the console, you must select an available subnet for your cluster. It is not marked as a required field. 

### Cluster provision with Ansible automation fails in proxy environment

An Automation template that is configured to automatically provision a managed cluster might fail when both of the following conditions are met: 

* The hub cluster has cluster-wide proxy enabled. 
* The Ansible Automation Platform can only be reached through the proxy.

### Cannot delete managed cluster namespace manually

You cannot delete the namespace of a managed cluster manually. The managed cluster namespace is automatically deleted after the managed cluster is detached. If you delete the managed cluster namespace manually before the managed cluster is detached, the managed cluster shows a continuous terminating  status after you delete the managed cluster. To delete this terminating managed cluster, manually remove the finalizers from the managed cluster that you detached.

### Automatic secret updates for provisioned clusters is not supported

When you change your cloud provider access key on the cloud provider side, you also need to update the corresponding credential for this cloud provider on the console of multicluster engine operator. This is required when your credentials expire on the cloud provider where the managed cluster is hosted and you try to delete the managed cluster.

### Process to destroy a cluster does not complete

When you destroy a managed cluster, the status continues to display `Destroying` after one hour, and the cluster is not destroyed. To resolve this issue complete the following steps:

1. Manually ensure that there are no orphaned resources on your cloud, and that all of the provider resources that are associated with the managed cluster are cleaned up.
2. Open the `ClusterDeployment` information for the managed cluster that is being removed by entering the following command:

   ```
   oc edit clusterdeployment/<mycluster> -n <namespace>
   ```

   Replace `_mycluster_` with the name of the managed cluster that you are destroying.

   Replace `_namespace_` with the namespace of the managed cluster.
3. Remove the `hive.openshift.io/deprovision` finalizer to forcefully stop the process that is trying to clean up the cluster resources in the cloud.
4. Save your changes and verify that `ClusterDeployment` is gone.
5. Manually remove the namespace of the managed cluster by running the following command:

   ```
   oc delete ns <namespace>
   ```

   Replace `_namespace_` with the namespace of the managed cluster.

### Cannot upgrade OpenShift Container Platform managed clusters on OpenShift Container Platform Dedicated with the console

You cannot use the Red Hat Advanced Cluster Management console to upgrade OpenShift Container Platform managed clusters that are in the OpenShift Container Platform Dedicated environment.

### Work manager add-on search details

The search details page for a certain resource on a certain managed cluster might fail. You must ensure that the work-manager add-on in the managed cluster is in `Available` status before you can search.

### Non-OpenShift Container Platform managed clusters require _ManagedServiceAccount_ or _LoadBalancer_ for pod logs 

The `ManagedServiceAccount` and cluster proxy add-ons are enabled by default in Red Hat Advanced Cluster Management version 2.10 and newer. If the add-ons are disabled after upgrading, you must enable the `ManagedServiceAccount` and cluster proxy add-ons manually to use the pod log feature on non-OpenShift Container Platform managed clusters.

See [ManagedServiceAccount add-on](../../clusters/install_upgrade/adv_config_install.adoc#serviceaccount-addon-intro) to learn how to enable `ManagedServiceAccount` and see [Using cluster proxy add-ons](../../clusters/cluster_lifecycle/cluster_proxy_addon.adoc#cluster-proxy-addon) to learn how to enable a cluster proxy add-on.

### OpenShift Container Platform 4.10.z does not support hosted control plane clusters with proxy configuration

When you create a hosting service cluster with a cluster-wide proxy configuration on OpenShift Container Platform 4.10.z, the `nodeip-configuration.service` service does not start on the worker nodes.

### Client cannot reach iPXE script

iPXE is an open source network boot firmware. See [iPXE](https://ipxe.org/) for more details.

When booting a node, the URL length limitation in some DHCP servers cuts off the `ipxeScript` URL in the `InfraEnv` custom resource definition, resulting in the following error message in the console:

`no bootable devices`

To work around the issue, complete the following steps:

1. Apply the `InfraEnv` custom resource definition when using an assisted installation to expose the `bootArtifacts`, which might resemble the following file:

   ```
   status:
     agentLabelSelector:
       matchLabels:
         infraenvs.agent-install.openshift.io: qe2
     bootArtifacts:
       initrd: https://assisted-image-service-multicluster-engine.redhat.com/images/0000/pxe-initrd?api_key=0000000&arch=x86_64&version=4.11
       ipxeScript: https://assisted-service-multicluster-engine.redhat.com/api/assisted-install/v2/infra-envs/00000/downloads/files?api_key=000000000&file_name=ipxe-script
       kernel: https://mirror.openshift.com/pub/openshift-v4/x86_64/dependencies/rhcos/4.12/latest/rhcos-live-kernel-x86_64
       rootfs: https://mirror.openshift.com/pub/openshift-v4/x86_64/dependencies/rhcos/4.12/latest/rhcos-live-rootfs.x86_64.img
   ```
2. Create a proxy server to expose the `bootArtifacts` with short URLs.
3. Copy the `bootArtifacts` and add them them to the proxy by running the following commands:

   ```
   for artifact in oc get infraenv qe2 -ojsonpath="{.status.bootArtifacts}" | jq ". | keys[]" | sed "s/\"//g"
   do curl -k oc get infraenv qe2 -ojsonpath="{.status.bootArtifacts.${artifact}}"` -o $artifact 
   ```
4. Add the `ipxeScript` artifact proxy URL to the `bootp` parameter in `libvirt.xml`.

### Cannot delete _ClusterDeployment_ after upgrading Red Hat Advanced Cluster Management

If you are using the removed BareMetalAssets API in Red Hat Advanced Cluster Management 2.6, the `ClusterDeployment` cannot be deleted after upgrading to Red Hat Advanced Cluster Management 2.7 because the BareMetalAssets API is bound to the `ClusterDeployment`.

To work around the issue, run the following command to remove the `finalizers` before upgrading to Red Hat Advanced Cluster Management 2.7:

```
oc patch clusterdeployment <clusterdeployment-name> -p '{"metadata":{"finalizers":null}}' --type=merge 
```

### Managed cluster stuck in _Pending_ status after deployment

The converged flow is the default process of provisioning. When you use the `BareMetalHost` resource for the Bare Metal Operator (BMO) to connect your host to a live ISO, the Ironic Python Agent does the following actions:

* It runs the steps in the Bare Metal installer-provisioned-infrastructure. 
* It starts the {ai} agent, and the agent handles the rest of the install and provisioning process. 

If the {ai} agent starts slowly and you deploy a managed cluster, the managed cluster might become stuck in the `Pending` status and not have any agent resources. You can work around the issue by disabling the converged flow. 

**Important:** When you disable the converged flow, only the {ai} agent runs in the live ISO, reducing the number of open ports and disabling any features you enabled with the  Ironic Python Agent agent, including the following:

* Pre-provisioning disk cleaning 
* iPXE boot firmware 
* BIOS configuration 

To decide what port numbers you want to enable or disable without disabling the converged flow, see [Network configuration](../../clusters/about/mce_networking.adoc#mce-network-configuration). 

To disable the converged flow, complete the following steps:

1. Create the following ConfigMap on the hub cluster:

   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: my-assisted-service-config
     namespace: multicluster-engine
   data:
     ALLOW_CONVERGED_FLOW: "false" <1> 
   ```
   1. When you set the parameter value to "false", you also disable any features enabled by the Ironic Python Agent. 
2. Apply the ConfigMap by running the following command:

   ```
   oc annotate --overwrite AgentServiceConfig agent unsupported.agent-install.openshift.io/assisted-service-configmap=my-assisted-service-config
   ```

### ManagedClusterSet API specification limitation

The `selectorType: LaberSelector` setting is not supported when using the [Clustersets API](../../apis/clusterset.json.adoc#clustersets-api). The `selectorType: ExclusiveClusterSetLabel` setting is supported.

### The Cluster curator does not support OpenShift Container Platform Dedicated clusters

When you upgrade an OpenShift Container Platform Dedicated cluster by using the `ClusterCurator` resource, the upgrade fails because the Cluster curator does not support OpenShift Container Platform Dedicated clusters.

### Custom ingress domain is not applied correctly

You can specify a custom ingress domain by using the `ClusterDeployment` resource while installing a managed cluster, but the change is only applied after the installation by using the `SyncSet` resource. As a result, the `spec` field in the `clusterdeployment.yaml` file displays the custom ingress domain you specified, but the `status` still displays the default domain.

### `ManagedClusterAddon` status becomes stuck

If you define configurations in the `ManagedClusterAddon` to override some configurations in the `ClusterManagementAddon`, the `ManagedClusterAddon` might become stuck at the following status:

```bash
progressing... mca and work configs mismatch
```

When you check the `ManagedClusterAddon` status, a part of the configurations has an empty `spec` hash, even if the configurations exist. See the following example:

```yaml
status:
  conditions:
  - lastTransitionTime: "2024-09-09T16:08:42Z"
    message: progressing... mca and work configs mismatch
    reason: Progressing
    status: "True"
    type: Progressing
...
  configReferences:
  - desiredConfig:
      name: deploy-config
      namespace: open-cluster-management-hub
      specHash: b81380f1f1a1920388d90859a5d51f5521cecd77752755ba05ece495f551ebd0
    group: addon.open-cluster-management.io
    lastObservedGeneration: 1
    name: deploy-config
    namespace: open-cluster-management-hub
    resource: addondeploymentconfigs
  - desiredConfig:
      name: cluster-proxy
      specHash: ""
    group: proxy.open-cluster-management.io
    lastObservedGeneration: 1
    name: cluster-proxy
    resource: managedproxyconfigurations 
```

To resolve the issue, delete the `ManagedClusterAddon` by running the following command to reinstall and recover the `ManagedClusterAddon`. Replace `<cluster-name>` with the `ManagedClusterAddon` namespace. Replace `<addon-name>` with the `ManagedClusterAddon` name:

```bash
oc -n <cluster-name> delete managedclusteraddon <addon-name>
```

## Central infrastructure management

### Cluster provisioning with {infra} fails

When creating OpenShift Container Platform clusters by using the {infra}, the file name of the ISO image might be too long. The long image name causes the image provisioning and the cluster provisioning to fail. To determine if this is the problem, complete the following steps: 

1. View the bare metal host information for the cluster that you are provisioning by running the following command: 

   ```
   oc get bmh -n <cluster_provisioning_namespace>
   ```
2. Run the `describe` command to view the error information:

   ```
   oc describe bmh -n <cluster_provisioning_namespace> <bmh_name>
   ```
3. An error similar to the following example indicates that the length of the filename is the problem: 

   ```
   Status:
     Error Count:    1
     Error Message:  Image provisioning failed: ... [Errno 36] File name too long ...
   ```

If this problem occurs, it is typically on the following versions of OpenShift Container Platform, because the {infra} was not using image service:

* 4.8.17 and earlier
* 4.9.6 and earlier

To avoid this error, upgrade your OpenShift Container Platform to version 4.8.18 or later, or 4.9.7 or later.

### Cannot use host inventory to boot with the discovery image and add hosts automatically

You cannot use a host inventory, or `InfraEnv` custom resource, to both boot with the discovery image and add hosts automatically. If you used your previous `InfraEnv` resource for the `BareMetalHost` resource, and you want to boot the image yourself, you can work around the issue by creating a new `InfraEnv` resource.

### A single-node OpenShift cluster installation requires a matching OpenShift Container Platform with {infra}

If you want to install a single-node OpenShift cluster with an Red Hat OpenShift Container Platform version before 4.16, your `InfraEnv` custom resource and your booted host must use the same OpenShift Container Platform version that you are using to install the single-node OpenShift cluster. The installation fails if the versions do not match.

To work around the issue, edit your `InfraEnv` resource before you boot a host with the Discovery ISO, and include the following content:

```yaml
apiVersion: agent-install.openshift.io/v1beta1
kind: InfraEnv
spec:
  osImageVersion: 4.15
```

The `osImageVersion` field must match the Red Hat OpenShift Container Platform cluster version that you want to install.

### _tolerations_ and _nodeSelector_ settings do not affect the _managed-serviceaccount_ agent

The `tolerations` and `nodeSelector` settings configured on the `MultiClusterEngine` and `MultiClusterHub` resources do not affect the `managed-serviceaccount` agent deployed on the local cluster. The `managed-serviceaccount` add-on is not always required on the local cluster.

If the `managed-serviceaccount` add-on is required, you can work around the issue by completing the following steps:

1. Create the `addonDeploymentConfig` custom resource.
2. Set the `tolerations` and `nodeSelector` values for the local cluster and `managed-serviceaccount` agent.
3. Update the `managed-serviceaccount` `ManagedClusterAddon` in the local cluster namespace to use the `addonDeploymentConfig` custom resource you created.

See [Configuring nodeSelectors and tolerations for klusterlet add-ons](../../add-ons/configure_nodeselector_tolerations_addons.adoc#configure-nodeselector-tolerations-addons) to learn more about how to use the `addonDeploymentConfig` custom resource to configure `tolerations` and `nodeSelector` for add-ons.
