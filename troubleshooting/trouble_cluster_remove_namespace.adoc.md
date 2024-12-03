# Namespace remains after deleting a cluster

When you remove a managed cluster, the namespace is normally removed as part of the cluster removal process. In rare cases, the namespace remains with some artifacts in it. In that case, you must manually remove the namespace.

## Symptom: Namespace remains after deleting a cluster

After removing a managed cluster, the namespace is not removed.

## Resolving the problem: Namespace remains after deleting a cluster

Complete the following steps to remove the namespace manually:

1. Run the following command to produce a list of the resources that remain in the &lt;cluster_name> namespace:

   ```
   oc api-resources --verbs=list --namespaced -o name | grep -E '^secrets|^serviceaccounts|^managedclusteraddons|^roles|^rolebindings|^manifestworks|^leases|^managedclusterinfo|^appliedmanifestworks'|^clusteroauths' | xargs -n 1 oc get --show-kind --ignore-not-found -n <cluster_name>
   ```

   Replace `cluster_name` with the name of the namespace for the cluster that you attempted to remove.
2. Delete each identified resource on the list that does not have a status of `Delete` by entering the following command to edit the list:

   ```
   oc edit <resource_kind> <resource_name> -n <namespace>
   ```

   Replace `resource_kind` with the kind of the resource.
   Replace `resource_name` with the name of the resource.
   Replace `namespace` with the name of the namespace of the resource.
3. Locate the `finalizer` attribute in the in the metadata.
4. Delete the non-Kubernetes finalizers by using the vi editor `dd` command. 
5. Save the list and exit the `vi` editor by entering the `:wq` command.
6. Delete the namespace by entering the following command:

   ```
   oc delete ns <cluster-name>
   ```

   Replace `cluster-name` with the name of the namespace that you are trying to delete.
