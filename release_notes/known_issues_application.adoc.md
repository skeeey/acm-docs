# Application known issues and limitations

Review the known issues for application management. The following list contains known issues for this release, or known issues that continued from the previous release. 

For your Red Hat OpenShift Container Platform cluster, see [OpenShift Container Platform known issues](https://docs.redhat.com/documentation/en-us/openshift_container_platform/4.15/html/release_notes/#ocp-4-15-known-issues). 

For more about deprecations and removals, see [Deprecations and removals for Red Hat Advanced Cluster Management](../release_notes/acm_deprecate_remove.adoc#deprecations-removals-acm).

See the following known issues for the Application lifecycle component.

## Application topology displays invalid expression 

When you use the `Exist` or `DoesNotExist` operators in the `Placement` resource, the application topology node details display the expressions as `#invalidExpr`. This display is wrong, and the expression is still valid and works in the `Placement` resource. To workaround this issue, edit the expression inside the `Placement` resource YAML. 

## Editing subscription applications with _PlacementRule_ does not display the subscription YAML in editor

After you create a subscription application that references a `PlacementRule` resource, the subscription YAML does not display in the YAML editor in the console. Use your terminal to edit your subscription YAML file.

## Helm Chart with secret dependencies cannot be deployed by the Red Hat Advanced Cluster Management subscription 

Using Helm Chart, you can define privacy data in a Kubernetes secret and refer to this secret within the `value.yaml` file of the Helm Chart.  

The username and password are given by the referred Kubernetes secret resource `dbsecret`. For example, see the following sample `value.yaml` file: 

```yaml
credentials:
  secretName: dbsecret
  usernameSecretKey: username
  passwordSecretKey: password
```

The Helm Chart with secret dependencies is only supported in the Helm binary CLI. It is not supported in the operator SDK Helm library. The Red Hat Advanced Cluster Management subscription controller applies the operator SDK Helm library to install and upgrade the Helm Chart. Therefore, the Red Hat Advanced Cluster Management subscription cannot deploy the Helm Chart with secret dependencies. 

## Topology does not correctly display for Argo CD pull model `ApplicationSet` application 

When you use the Argo CD pull model to deploy `ApplicationSet` applications and the application resource names are customized, the resource names might appear different for each cluster. When this happens, the topology does not display your application correctly.

## Local cluster is excluded as a managed cluster for pull model

The hub cluster application set deploys to target managed clusters, but the local cluster, which is a managed hub cluster, is excluded as a target managed cluster.

As a result, if the Argo CD application is propagated to the local cluster by the Argo CD pull model, the local cluster Argo CD application is not cleaned up, even though the local cluster is removed from the placement decision of the Argo CD `ApplicationSet` resource.

To work around the issue and clean up the local cluster Argo CD application, remove the `skip-reconcile` annotation from the local cluster Argo CD application. See the following annotation:

```yaml
annotations:
    argocd.argoproj.io/skip-reconcile: "true"
```

Additionally, if you manually refresh the pull model Argo CD application in the **Applications** section of the Argo CD console, the refresh is not processed and the **REFRESH** button in the Argo CD console is disabled.

To work around the issue, remove the `refresh` annotation from the Argo CD application. See the following annotation:

```yaml
annotations:
    argocd.argoproj.io/refresh: normal 
```

## Argo CD controller and the propagation controller might reconcile simultaneously

Both the Argo CD controller and the propagation controller might reconcile on the same application resource and cause the duplicate instances of application deployment on the managed clusters, but from the different deployment models.

For deploying applications by using the pull model, the Argo CD controllers ignore these application resources when the Argo CD `argocd.argoproj.io/skip-reconcile` annotation is added to the template section of the `ApplicationSet`. 

The `argocd.argoproj.io/skip-reconcile` annotation is only available in the GitOps operator version 1.9.0, or later. To prevent conflicts, wait until the hub cluster and all the managed clusters are upgraded to GitOps operator version 1.9.0 before implementing the pull model. 

## Resource fails to deploy

All the resources listed in the `MulticlusterApplicationSetReport` are actually deployed on the managed clusters. If a resource fails to deploy, the resource is not included in the resource list, but the cause is listed in the error message.

## Resource allocation might take several minutes

For large environments with over 1000 managed clusters and Argo CD application sets that are deployed to hundreds of managed clusters, Argo CD application creation on the hub cluster might take several minutes. You can set the `requeueAfterSeconds` to `zero` in the `clusterDecisionResource` generator of the application set, as it is displayed in the following example file: 

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: cm-allclusters-app-set
  namespace: openshift-gitops
spec:
  generators:
  - clusterDecisionResource:
      configMapRef: ocm-placement-generator
      labelSelector:
        matchLabels:
          cluster.open-cluster-management.io/placement: app-placement
      requeueAfterSeconds: 0
```

## Application ObjectBucket channel type cannot use allow and deny lists

You cannot specify allow and deny lists with ObjectBucket channel type in the `subscription-admin` role. In other channel types, the allow and deny lists in the subscription indicates which Kubernetes resources can be deployed, and which Kubernetes resources should not be deployed.

### Argo Application cannot be deployed on 3.x OpenShift Container Platform managed clusters

Argo `ApplicationSet` from the console cannot be deployed on 3.x OpenShift Container Platform managed clusters because the `Infrastructure.config.openshift.io` API is not available on  on 3.x.

## Changes to the multicluster_operators_subscription image do not take effect automatically

The `application-manager` add-on that is running on the managed clusters is now handled by the subscription operator, when it was previously handled by the klusterlet operator. The subscription operator is not managed the `multicluster-hub`, so changes to the `multicluster_operators_subscription` image in the `multicluster-hub` image manifest ConfigMap do not take effect automatically.

If the image that is used by the subscription operator is overrided by changing the `multicluster_operators_subscription` image in the `multicluster-hub` image manifest ConfigMap, the `application-manager` add-on on the managed clusters does not use the new image until the subscription operator pod is restarted. You need to restart the pod.

## Policy resource not deployed unless by subscription administrator

The `policy.open-cluster-management.io/v1` resources are no longer deployed by an application subscription by default for Red Hat Advanced Cluster Management version 2.4.

A subscription administrator needs to deploy the application subscription to change this default behavior.

See [Creating an allow and deny list as subscription administrator](../applications/allow_deny.adoc) for information. `policy.open-cluster-management.io/v1` resources that were deployed by existing application subscriptions in previous Red Hat Advanced Cluster Management versions remain, but are no longer reconciled with the source repository unless the application subscriptions are deployed by a subscription administrator.

## Application Ansible hook stand-alone mode

Ansible hook stand-alone mode is not supported. To deploy Ansible hook on the hub cluster with a subscription, you might use the following subscription YAML:

```yaml
apiVersion: apps.open-cluster-management.io/v1
kind: Subscription
metadata:
  name: sub-rhacm-gitops-demo
  namespace: hello-openshift
annotations:
  apps.open-cluster-management.io/github-path: myapp
  apps.open-cluster-management.io/github-branch: master
spec:
  hooksecretref:
      name: toweraccess
  channel: rhacm-gitops-demo/ch-rhacm-gitops-demo
  placement:
     local: true
```
However, this configuration might never create the Ansible instance, since the `spec.placement.local:true` has the subscription running on `standalone` mode. You need to create the subscription in hub mode. 

1. Create a placement rule that deploys to `local-cluster`. See the following sample where `local-cluster: "true"` refers to your hub cluster:

+
```yaml
apiVersion: apps.open-cluster-management.io/v1
kind: PlacementRule
metadata: 
  name: <towhichcluster>
  namespace: hello-openshift
spec:
  clusterSelector:
    matchLabels:
      local-cluster: "true" 
```
1. Reference that placement rule in your subscription. See the following sample:

+
```yaml
apiVersion: apps.open-cluster-management.io/v1
kind: Subscription
metadata:
  name: sub-rhacm-gitops-demo
  namespace: hello-openshift
annotations:
  apps.open-cluster-management.io/github-path: myapp
  apps.open-cluster-management.io/github-branch: master
spec:
  hooksecretref:
      name: toweraccess
  channel: rhacm-gitops-demo/ch-rhacm-gitops-demo
  placement:
     placementRef:
        name: <towhichcluster>
        kind: PlacementRule
```

After applying both, you should see the Ansible instance created in your hub cluster.

## Application not deployed after an updated placement rule

If applications are not deploying after an update to a placement rule, verify that the `application-manager` pod is running.
The `application-manager` is the subscription container that needs to run on managed clusters.

You can run `oc get pods -n open-cluster-management-agent-addon |grep application-manager` to verify.

You can also search for `kind:pod cluster:yourcluster` in the console and see if the `application-manager` is running.

If you cannot verify, attempt to import the cluster again and verify again.

## Subscription operator does not create an SCC

Learn about Red Hat OpenShift Container Platform SCC at [Managing security context constraints](https://docs.redhat.com/en/documentation/openshift_container_platform/4.14/html-single/authentication_and_authorization/index#managing-pod-security-policies), which is an additional configuration required on the managed cluster.

Different deployments have different security context and different service accounts. The subscription operator cannot create an SCC CR automatically.. Administrators control permissions for pods. A Security Context Constraints (SCC) CR is required to enable appropriate permissions for the relative service accounts to create pods in the non-default namespace. To manually create an SCC CR in your namespace, complete the following steps:

1. Find the service account that is defined in the deployments. For example, see the following `nginx` deployments:

+
```
nginx-ingress-52edb
nginx-ingress-52edb-backend
```

+
. Create an SCC CR in your namespace to assign the required permissions to the service account or accounts. See the following example, where `kind: SecurityContextConstraints` is added:

+
```yaml
apiVersion: security.openshift.io/v1
 defaultAddCapabilities:
 kind: SecurityContextConstraints
 metadata:
   name: ingress-nginx
   namespace: ns-sub-1
 priority: null
 readOnlyRootFilesystem: false
 requiredDropCapabilities:
 fsGroup:
   type: RunAsAny
 runAsUser:
   type: RunAsAny
 seLinuxContext:
   type: RunAsAny
 users:
 - system:serviceaccount:my-operator:nginx-ingress-52edb
 - system:serviceaccount:my-operator:nginx-ingress-52edb-backend
```

## Application channels require unique namespaces

Creating more than one channel in the same namespace can cause errors with the hub cluster.

For instance, namespace `charts-v1` is used by the installer as a Helm type channel, so do not create any additional channels in `charts-v1`. Ensure that you create your channel in a unique namespace. All channels need an individual namespace, except GitHub channels, which can share a namespace with another GitHub channel.

## Ansible Automation Platform job fail

Ansible jobs fail to run when you select an incompatible option. Ansible Automation Platform only works when the `-cluster-scoped` channel options are chosen. This affects all components that need to perform Ansible jobs.

## Ansible Automation Platform operator access Ansible Automation Platform outside of a proxy

The Red Hat Ansible Automation Platform operator cannot access Ansible Automation Platform outside of a proxy-enabled OpenShift Container Platform cluster. To resolve, you can install the Ansible Automation Platform within the proxy. See install steps that are provided by Ansible Automation Platform.

## Application name requirements

An application name cannot exceed 37 characters. The application deployment displays the following error if the characters exceed this amount.

```yaml
status:
  phase: PropagationFailed
  reason: 'Deployable.apps.open-cluster-management.io "_long_lengthy_name_" is invalid: metadata.labels: Invalid value: "_long_lengthy_name_": must be no more than 63 characters/n'
```

## Application console table limitations

See the following limitations to various _Application_ tables in the console:

* From the _Applications_ table on the _Overview_ page and the _Subscriptions_ table on the _Advanced configuration_ page, the _Clusters_ column displays a count of clusters where application resources are deployed. Since applications are defined by resources on the local cluster, the local cluster is included in the search results, whether actual application resources are deployed on the local cluster or not.
* From the _Advanced configuration_ table for _Subscriptions_, the _Applications_ column displays the total number of applications that use that subscription, but if the subscription deploys child applications, those are included in the search result, as well.
* From the _Advanced configuration_ table for _Channels_, the _Subscriptions_ column displays the total number of subscriptions on the local cluster that use that channel, but this does not include subscriptions that are deployed by other subscriptions, which are included in the search result.

## No Application console topology filtering

The _Console_ and _Topology_ for _Application_ changes for the 2.12. There is no filtering capability from the console Topology page.

## Allow and deny list does not work in Object storage applications

The `allow` and `deny` list feature does not work in Object storage application subscriptions.
