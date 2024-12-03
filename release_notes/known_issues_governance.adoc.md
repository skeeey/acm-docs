# Governance known issues

Review the known issues for Governance. The following list contains known issues for this release, or known issues that continued from the previous release. 

For your Red Hat OpenShift Container Platform cluster, see [OpenShift Container Platform known issues](https://docs.redhat.com/documentation/en-us/openshift_container_platform/4.15/html/release_notes/#ocp-4-15-known-issues).

For more about deprecations and removals, see [Deprecations and removals for Red Hat Advanced Cluster Management](../release_notes/acm_deprecate_remove.adoc#deprecations-removals-acm).

## Configuration policy listed complaint when namespace is stuck in _Terminating_ state

When you have a configuration policy that is configured with `mustnothave` for the `complianceType` parameter and `enforce` for the `remediationAction` parameter, the policy is listed as compliant when a deletion request is made to the Kubernetes API. Therefore, the Kubernetes object can be stuck in a `Terminating` state while the policy is listed as compliant.

## Operators deployed with policies do not support ARM

While installation into an ARM environment is supported, operators that are deployed with policies might not support ARM environments. The following policies that install operators do not support ARM environments:

* [Red Hat Advanced Cluster Management policy for the Quay Container Security Operator](https://github.com/stolostron/policy-collection/blob/main/stable/SI-System-and-Information-Integrity/policy-imagemanifestvuln.yaml)
* [Red Hat Advanced Cluster Management policy for the Compliance Operator](https://github.com/stolostron/policy-collection/blob/main/stable/CA-Security-Assessment-and-Authorization/policy-compliance-operator-install.yaml)

## ConfigurationPolicy custom resource definition is stuck in terminating

When you remove the `config-policy-controller` add-on from a managed cluster by disabling the policy controller in the `KlusterletAddonConfig` or by detaching the cluster, the `ConfigurationPolicy` custom resource definition might get stuck in a terminating state. If the `ConfigurationPolicy` custom resource definition is stuck in a terminating state, new policies might not be added to the cluster if the add-on is reinstalled later. You can also receive the following error:

```
template-error; Failed to create policy template: create not allowed while custom resource definition is terminating
```

Use the following command to check if the custom resource definition is stuck: 

```
oc get crd configurationpolicies.policy.open-cluster-management.io -o=jsonpath='{.metadata.deletionTimestamp}'
```

If a deletion timestamp is on the resource, the custom resource definition is stuck. To resolve the issue, remove all finalizers from configuration policies that remain on the cluster. Use the following command on the managed cluster and replace `<cluster-namespace>` with the managed cluster namespace:

```
oc get configurationpolicy -n <cluster-namespace> -o name | xargs oc patch -n <cluster-namespace> --type=merge -p '{"metadata":{"finalizers": []}}'
```

The configuration policy resources are automatically removed from the cluster and the custom resource definition exits its terminating state. If the add-on has already been reinstalled, the custom resource definition is recreated automatically without a deletion timestamp. 

## Policy status shows repeated updates when enforced

If a policy is set to `remediationAction: enforce` and is repeatedly updated, the Red Hat Advanced Cluster Management console shows repeated violations with successful updates. See the following two possible causes and solutions for the error:

* Another controller or process is also updating the object with different values.

  To resolve the issue, disable the policy and compare the differences between `objectDefinition` in the policy and the object on the managed cluster. If the values are different, another controller or process might be updating them. Check the `metadata` of the object to help identify why the values are different.
* The `objectDefinition` in the `ConfigurationPolicy` does not match because of Kubernetes processing the object when the policy is applied.

  To resolve the issue, disable the policy and compare the differences between `objectDefinition` in the policy and the object on the managed cluster. If the keys are different or missing, Kubernetes might have processed the keys before applying them to the object, such as removing keys containing default or empty values.

## Duplicate policy template names create inconstistent results

When you create a policy with identical policy template names, you receive inconsistent results that are not detected, but you might not know the cause. For example, defining a policy with multiple configuration policies named `create-pod` causes inconsistent results. **Best practice:** Avoid using duplicate names for policy templates.

## Database and policy compliance history API outage

There is built-in resilience for database and policy compliance history API outages, however, any compliance events that cannot be recorded by a managed cluster are queued in memory until they are successfully recorded. This means that if there is an outage and the `governance-policy-framework` pod on the managed cluster restarts, all queued compliance events are lost.

If you create or update a new policy during a database outage, any compliance events sent for this new policy cannot be recorded since the mapping of policies to database IDs cannot be updated. When the database is back online, the mapping is automatically updated and future compliance events from those policies are recorded.

## PostgreSQL data loss

If there is data loss to the PostgreSQL server such as restoring to a backup without the latest data, you must restart the governance policy propagator on the Red Hat Advanced Cluster Management hub cluster so that it can update the mapping of policies to database IDs. Until you restart the governance policy propagator, new compliance events associated with policies that once existed in the database are no longer recorded. 

To restart the governance policy propagator, run the following command on the Red Hat Advanced Cluster Management hub cluster:

```bash
oc -n open-cluster-management rollout restart deployment/grc-policy-propagator
```

## Kyverno policies no longer report a status for the latest version 

Kyverno policies generated by the Policy Generator report the following message in your Red Hat Advanced Cluster Management cluster:

```
violation - couldn't find mapping resource with kind ClusterPolicyReport, please check if you have CRD deployed;
violation - couldn't find mapping resource with kind PolicyReport, please check if you have CRD deployed
```

The cause is that the `PolicyReport` API version is incorrect in the generator and does not match what Kyverno has deployed.
