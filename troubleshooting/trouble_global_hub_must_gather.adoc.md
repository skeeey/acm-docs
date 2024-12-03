# Troubleshooting with the _must-gather_ command 

## Symptom: Errors with multicluster global hub

You might experience various errors with multicluster global hub. You can run the `must-gather` command for troubleshooting issues with multicluster global hub.

## Resolving the problem: Running the _must-gather_ command for dubugging

Run the `must-gather` command to gather details, logs, and take steps in debugging issues. This debugging information is also useful when you open a support request. The `oc adm must-gather` CLI command collects the information from your cluster that is often needed for debugging issues, including:

* Resource definitions
* Service logs

### Prerequisites

You must meet the following prerequisites to run the `must-gather` command: 

* Access to the global hub and managed hub clusters as a user with the `cluster-admin` role.
* The OpenShift Container Platform CLI (oc) installed.

### Running the must-gather command

Complete the following procedure to collect information by using the must-gather command:

1. Learn about the `must-gather` command and install the prerequisites that you need by reading the [Gathering data about your cluster](https://docs.redhat.com/documentation/en-us/openshift_container_platform/4.14/html/support/gathering-cluster-data) in the OpenShift Container Platform documentation.
2. Log in to your global hub cluster. For the typical use case, run the following command while you are logged into your global hub cluster:

   ```
   oc adm must-gather --image=quay.io/stolostron/must-gather:SNAPSHOTNAME
   ```

   If you want to check your managed hub clusters, run the `must-gather` command on those clusters.
3. Optional: If you want to save the results in a the `SOMENAME` directory, you can run the following command instead of the one in the previous step:

   ```
   oc adm must-gather --image=quay.io/stolostron/must-gather:SNAPSHOTNAME --dest-dir=<SOMENAME> ; tar -cvzf <SOMENAME>.tgz <SOMENAME>
   ```

   You can specify a different name for the directory. 

   **Note:** The command includes the required additions to create a gzipped tarball file.

The following information is collected from the `must-gather` command: 

* Two peer levels: `cluster-scoped-resources` and `namespaces` resources.
* Sub-level for each: API group for the custom resource definitions for both cluster-scope and namespace-scoped resources.
* Next level for each: YAML file sorted by kind.
* For the global hub cluster, you can check the `PostgresCluster` and `Kafka` in the `namespaces` resources.
* For the global hub cluster, you can check the multicluster global hub related pods and logs in `pods` of `namespaces` resources.
* For the managed hub cluster, you can check the multicluster global hub agent pods and logs in `pods` of `namespaces` resources.
