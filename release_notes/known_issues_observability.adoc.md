# Observability known issues

Review the known issues for Red Hat Advanced Cluster Management for Kubernetes. The following list contains known issues for this release, or known issues that continued from the previous release. 

For your Red Hat OpenShift Container Platform cluster, see link:https://docs.redhat.com/documentation/en-us/openshift_container_platform/4.15/html/release_notes#ocp-4-15-known-issues [OpenShift Container Platform known issues].

For more about deprecations and removals, see [Deprecations and removals for Red Hat Advanced Cluster Management](../release_notes/acm_deprecate_remove.adoc#deprecations-removals-acm). 

## Grafana dashboard missing 

A Grafana dashboard might fail to load after you run the Grafana instance. Complete the following steps:

1. To verify whether a dashboard failed to load, check the logs by running the following command:

+
```bash
oc logs observability-grafana-68f8489659-m79rv -c grafana-dashboard-loader -n open-cluster-management-observability
...
E1017 12:55:24.532493 1 dashboard_controller.go:147] dashboard: sample-dashboard could not be created after retrying 40 times
```

1. To fix the dashboard failure, redeploy Grafana by scaling the number of replicas to `0`. The `multicluster-observability-operator` pod automatically scales the deployment to the desired number of replicas that is defined in the `MultiClusterObservability` resource. Run the following command:

+
```bash
oc scale deployment observability-grafana -n open-cluster-management-observability --replicas=0
```

1. To verify that the dashboard loads correctly after redployment, run the following command to check the logs of all Grafana pods and ensure no error message appears:

+
```bash
oc logs observability-grafana-68f8489659-h6jd9 -c grafana-dashboard-loader -n open-cluster-management-observability | grep "could not be created"
```

## Retention change causes data loss 

The default retention for all resolution levels, such as `retentionResolutionRaw`, `retentionResolution5m`, or `retentionResolution1h`, is 365 days (`365d`). This `365d` default retention means that the default retention for a 1 hour resolution has decreased from indefinite, `0d` to `365d`. This retention change might cause you to lose data. If you did not set an explicit value for the resolution retention in your `MultiClusterObservability` `spec.advanced.retentionConfig` parameter, you might lose data.  

For more information, see [Adding advanced configuration for retention](../observability/customize_observability.adoc#adding-advanced-config).

## Observatorium API gateway pods in a restored hub cluster might have stale tenant data

The Observatorium API gateway pods in a restored hub cluster might contain stale tenant data after a backup and restore procedure because of a Kubernetes limitation. See [Mounted ConfigMaps are updated automatically](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#mounted-configmaps-are-updated-automatically) for more about the limitation.

As a result, the Observatorium API and Thanos gateway rejects metrics from collectors, and the Red Hat Advanced Cluster Management Grafana dashboards do not display data.

See the following errors from the Observatorium API gateway pod logs:

```
level=error name=observatorium caller=logchannel.go:129 msg="failed to forward metrics" returncode="500 Internal Server Error" response="no matching hashring to handle tenant\n"
```

Thanos receives pods logs with the following errors:

```
caller=handler.go:551 level=error component=receive component=receive-handler tenant=xxxx err="no matching hashring to handle tenant" msg="internal server error"
```

See the following procedure to resolve this issue:

1. Scale down the `observability-observatorium-api` deployment instances from `N` to `0`.
2. Scale up the `observability-observatorium-api` deployment instances from `0` to `N`. 

**Note:** `N` = `2` by default, but might be greater than `2` in some custom configuration environments.

This restarts all Observatorium API gateway pods with the correct tenant information, and the data from collectors start displaying in Grafana in between 5-10 minutes.

## Permission to add _PrometheusRules_ and _ServiceMonitors_ in _openshift-monitoring_ namespace denied

Starting with Red Hat Advanced Cluster Management 2.9, you must use a label in your defined Red Hat Advanced Cluster Management hub cluster namespace. The label, `openshift.io/cluster-monitoring: "true"` causes the Cluster Monitoring Operator to scrape the namespace for metrics. 

When Red Hat Advanced Cluster Management 2.9 is deployed or an installation is upgraded to 2.9, the Red Hat Advanced Cluster Management Observability `ServiceMonitors` and `PrometheusRule` resources are no longer present in the `openshift-monitoring` namespace. 

## Lack of support for proxy settings

The Prometheus `AdditionalAlertManagerConfig` resource of the observability add-on does not support proxy settings. You must disable the observability alert forwarding feature. 

Complete the following steps to disable alert forwarding:

1. Go to the `MultiClusterObservability` resource.
2. Update the `mco-disabling-alerting` parameter value to `true`

The HTTPS proxy with a self-signed CA certificate is not supported. 

## Duplicate local-clusters on Service-level Overview dashboard

When various hub clusters deploy Red Hat Advanced Cluster Management observability using the same S3 storage, _duplicate_ `local-clusters` can be detected and displayed within the _Kubernetes/Service-Level Overview/API Server_ dashboard. The duplicate clusters affect the results within the following panels: _Top Clusters_, _Number of clusters that has exceeded the SLO_, and _Number of clusters that are meeting the SLO_. The `local-clusters` are unique clusters associated with the shared S3 storage. To prevent multiple `local-clusters` from displaying within the dashboard, it is recommended for each unique hub cluster to deploy observability with a S3 bucket specifically for the hub cluster.

## Observability endpoint operator fails to pull image

The observability endpoint operator fails if you create a pull-secret to deploy to the MultiClusterObservability CustomResource (CR) and there is no pull-secret in the `open-cluster-management-observability` namespace. When you import a new cluster, or import a Hive cluster that is created with Red Hat Advanced Cluster Management, you need to manually create a pull-image secret on the managed cluster.

For more information, see [Enabling observability](../observability/observability_enable.adoc#enabling-observability).

## There is no data from ROKS clusters

Red Hat Advanced Cluster Management observability does not display data from a ROKS cluster on some panels within built-in dashboards. This is because ROKS does not expose any API server metrics from servers they manage. The following Grafana dashboards contain panels that do not support ROKS clusters: `Kubernetes/API server`, `Kubernetes/Compute Resources/Workload`, `Kubernetes/Compute Resources/Namespace(Workload)`

## There is no etcd data from ROKS clusters

For ROKS clusters, Red Hat Advanced Cluster Management observability does not display data in the _etcd_ panel of the dashboard.

## Metrics are unavailable in the Grafana console

* Annotation query failed in the Grafana console: 

  When you search for a specific annotation in the Grafana console, you might receive the following error message due to an expired token: 

  `"Annotation Query Failed"`

  Refresh your browser and verify you are logged into your hub cluster.
* Error in _rbac-query-proxy_ pod:

  Due to unauthorized access to the `managedcluster` resource, you might receive the following error when you query a cluster or project:

  `no project or cluster found`

  Check the role permissions and update appropriately. See [Role-based access control](../access_control/rbac.adoc#role-based-access-control) for more information. 

## Prometheus data loss on managed clusters

By default, Prometheus on OpenShift uses ephemeral storage. Prometheus loses all metrics data whenever it is restarted.

When observability is enabled or disabled on OpenShift Container Platform managed clusters that are managed by Red Hat Advanced Cluster Management, the observability endpoint operator updates the `cluster-monitoring-config` `ConfigMap` by adding additional alertmanager configuration that restarts the local Prometheus automatically. 

## Error ingesting out-of-order samples

Observability `receive` pods report the following error message:

```
Error on ingesting out-of-order samples
```

The error message means that the time series data sent by a managed cluster, during a metrics collection interval is older than the time series data it sent in the previous collection interval. When this problem happens, data is discarded by the Thanos receivers and this might create a gap in the data shown in Grafana dashboards. If the error is seen frequently, it is recommended to increase the metrics collection interval to a higher value. For example, you can increase the interval to 60 seconds.

The problem is only noticed when the time series interval is set to a lower value, such as 30 seconds. Note, this problem is not seen when the metrics collection interval is set to the default value of 300 seconds.

## Grafana deployment fails after upgrade

If you have a `grafana-dev` instance deployed in earlier versions before 2.6, and you upgrade the environment to 2.6, the `grafana-dev` does not work. You must delete the existing `grafana-dev` instance by running the following command:

```
./setup-grafana-dev.sh --clean
```

Recreate the instance with the following command:

```
./setup-grafana-dev.sh --deploy
```

## _klusterlet-addon-search_ pod fails

The `klusterlet-addon-search` pod fails because the memory limit is reached. You must update the memory request and limit by customizing the `klusterlet-addon-search` deployment on your managed cluster. Edit the `ManagedclusterAddon` custom resource named `search-collector`, on your hub cluster. Add the following annotations to the `search-collector` and update the memory, `addon.open-cluster-management.io/search_memory_request=512Mi` and `addon.open-cluster-management.io/search_memory_limit=1024Mi`.

For example, if you have a managed cluster named `foobar`, run the following command to change the memory request to `512Mi` and the memory limit to `1024Mi`:

```
oc annotate managedclusteraddon search-collector -n foobar \
addon.open-cluster-management.io/search_memory_request=512Mi \
addon.open-cluster-management.io/search_memory_limit=1024Mi
```

## Enabling _disableHubSelfManagement_ causes empty list in Grafana dashboard

The Grafana dashboard shows an empty label list if the `disableHubSelfManagement` parameter is set to `true` in the `mulitclusterengine` custom resource. You must set the parameter to `false` or remove the parameter to see the label list. See [disableHubSelfManagement](../install/adv_config_install.adoc#disable-hub-self-management) for more details.

### Endpoint URL cannot have fully qualified domain names (FQDN)

When you use the FQDN or protocol for the `endpoint` parameter, your observability pods are not enabled. The following error message is displayed:

```bash
Endpoint url cannot have fully qualified paths
```

Enter the URL without the protocol. Your `endpoint` value must resemble the following URL for your secrets:

```bash
endpoint: example.com:443
```

### Grafana downsampled data mismatch

When you attempt to query historical data and there is a discrepancy between the calculated step value and downsampled data, the result is empty. For example, if the calculated step value is `5m` and the downsampled data is in a one-hour interval, data does not appear from Grafana.

This discrepancy occurs because a URL query parameter must be passed through the Thanos Query front-end data source. Afterwards, the URL query can perform additional queries for other downsampling levels when data is missing.

You must manually update the Thanos Query front-end data source configuration. Complete the following steps:

1. Go to the Query front-end data source.
2. To update your query parameters, click the _Misc_ section.
3. From the _Custom query parameters_ field, select **`max_source_resolution=auto`**.
4. To verify that the data is displayed, refresh your Grafana page. 

Your query data appears from the Grafana dashboard.

## Metrics collector does not detect proxy configuration

A proxy configuration in a managed cluster that you configure by using the `addonDeploymentConfig` is not detected by the metrics collector. As a workaround, you can enable the proxy by removing the managed cluster `ManifestWork`. Removing the `ManifestWork` forces the changes in the `addonDeploymentConfig` to be applied.

## Limitations when using custom managed cluster Observatorium API or Alertmanager URLs

Custom Observatorium API and Alertmanager URLs only support intermediate components with TLS passthrough. If both custom URLs are pointing to the same intermediate component, you must use separate sub-domains because OpenShift Container Platform routers do not support two separate route objects with the same host.

### Search does not display node information from the managed cluster

Search maps RBAC for resources in the hub cluster. Depending on user RBAC settings, users might not see node data from the managed cluster. Results from search might be different from what is displayed on the _Nodes_ page for a cluster.
