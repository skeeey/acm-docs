# Console known issues

Review the known issues for the console. The following list contains known issues for this release, or known issues that continued from the previous release. 

For your Red Hat OpenShift Container Platform cluster, see [OpenShift Container Platform known issues](https://docs.redhat.com/documentation/en-us/openshift_container_platform/4.15/html/release_notes#ocp-4-15-known-issues). 

For more about deprecations and removals, see [Deprecations and removals for Red Hat Advanced Cluster Management](../release_notes/acm_deprecate_remove.adoc#deprecations-removals-acm).

## Cannot upgrade OpenShift Dedicated in console

From the console you can request an upgrade for OpenShift Dedicated clusters, but the upgrade fails with the `Cannot upgrade non openshift cluster` error message. Currently there is no workaround. 

## Search PostgreSQL pod is in CrashLoopBackoff state

The `search-postgres` pod is in `CrashLoopBackoff` state. If Red Hat Advanced Cluster Management is deployed in a cluster with nodes that have the `hugepages` parameter enabled and the `search-postgres` pod gets scheduled in these nodes, then the pod does not start.

Complete the following steps to increase the memory of the `search-postgres` pod:

1. Pause the `search-operator` pod with the following command:

   ```bash
   oc annotate search search-v2-operator search-pause=true
   ```
2. Update the `search-postgres` deployment with a limit for the `hugepages` parameter. Run the following command to set the `hugepages` parameter to `512Mi`:

   ```bash
   oc patch deployment search-postgres --type json -p '[{"op": "add", "path": "/spec/template/spec/containers/0/resources/limits/hugepages-2Mi", "value":"512Mi"}]'
   ```
3. Before you verify the memory usage for the pod, make sure your `search-postgres` pod is in the `Running` state. Run the following command:

   ```bash
   oc get pod <your-postgres-pod-name>  -o jsonpath="Status: {.status.phase}"
   ```
4. Run the following command to verify the memory usage of the `search-postgres` pod:

   ```bash
   oc get pod <your-postgres-pod-name> -o jsonpath='{.spec.containers[0].resources.limits.hugepages-2Mi}'
   ```

The following value appears, `512Mi`.

## Cannot edit namespace bindings for cluster set

When you edit namespace bindings for a cluster set with the `admin` role or `bind` role, you might encounter an error that resembles the following message:

`ResourceError: managedclustersetbindings.cluster.open-cluster-management.io "<cluster-set>" is forbidden: User "<user>" cannot create/delete resource "managedclustersetbindings" in API group "cluster.open-cluster-management.io" in the namespace "<namespace>".`

To resolve the issue, make sure you also have permission to create or delete a `ManagedClusterSetBinding` resource in the namespace you want to bind. The role bindings only allow you to bind the cluster set to the namespace.

## Horizontal scrolling does not work after provisioning hosted control plane cluster

After provisioning a hosted control plane cluster, you might not be able to scroll horizontally in the cluster overview of the Red Hat Advanced Cluster Management console if the `ClusterVersionUpgradeable` parameter is too long. You cannot view the hidden data as a result.

To work around the issue, zoom out by using your browser zoom controls, increase your Red Hat Advanced Cluster Management console window size, or copy and paste the text to a different location.

## _EditApplicationSet_ expand feature repeats

When you add multiple label expressions or attempt to enter your cluster selector for your `ApplicationSet`, you might receive the following message repeatedly,  "Expand to enter expression". You can enter your cluster selection despite this issue.

## Unable to log out from Red Hat Advanced Cluster Management

When you use an external identity provider to log in to Red Hat Advanced Cluster Management, you might not be able to log out of Red Hat Advanced Cluster Management. This occurs when you use Red Hat Advanced Cluster Management, installed with IBM Cloud and Keycloak as the identity providers.

You must log out of the external identity provider before you attempt to log out of Red Hat Advanced Cluster Management.

## Issues with entering the _cluster-ID_ in the OpenShift Cloud Manager console

If you did not access the `cluster-ID` in the OpenShift Cloud Manager console, you can still get a description of your OpenShift Service on AWS `cluster-ID` from the terminal. You need the OpenShift Service on AWS command line interface. See [Getting started with the OpenShift Service on AWS CLI](https://docs.redhat.com/en/documentation/red_hat_openshift_service_on_aws/4/html/rosa_cli/rosa-get-started-cli#rosa-get-started-cli) documentation.

To get the `cluster-ID`, run the following command from the OpenShift Service on AWS command line interface:

```bash
rosa describe cluster --cluster=<cluster-name> | grep -o ’^ID:.*
```
