# Installation known issues

Review the known issues for installing and upgrading. The following list contains known issues for this release, or known issues that continued from the previous release. 

For your Red Hat OpenShift Container Platform cluster, see [OpenShift Container Platform known issues](https://docs.redhat.com/documentation/en-us/openshift_container_platform/4.15/html/release_notes/#ocp-4-15-known-issues). 

For more about deprecations and removals, see [Deprecations and removals for Red Hat Advanced Cluster Management](../release_notes/acm_deprecate_remove.adoc#deprecations-removals-acm).

## Uninstalling and reinstalling earlier versions with an upgrade can fail

Uninstalling Red Hat Advanced Cluster Management from OpenShift Container Platform can cause issues if you later want to install earlier versions and then upgrade. For instance, when you uninstall Red Hat Advanced Cluster Management, then install an earlier version of Red Hat Advanced Cluster Management and upgrade that version, the upgrade might fail. The upgrade fails if the custom resources were not removed.

Follow the [Cleaning up artifacts before reinstalling](../install/cleanup_reinstall.adoc#cleanup-reinstall) procedure to prevent this problem.

## Infrastructure operator error with ARM converged flow

When you install the `infrastructure-operator`, converged flow with ARM does not work. Set `ALLOW_CONVERGED_FLOW` to `false` to resolve this issue.

1. Run the following command to create a `ConfigMap` resource:

+
```
oc create -f
```

1. Apply your file by running `oc apply -f`. See the following file sample with `ALLOW_CONVERGED_FLOW` set to `false`:

+
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-assisted-service-config
  namespace: assisted-installer
data:
  ALLOW_CONVERGED_FLOW: false
```

1. Annotate the `agentserviceconfig` with the following command:

+
```
oc annotate --overwrite AgentServiceConfig agent unsupported.agent-install.openshift.io/assisted-service-configmap=my-assisted-service-config
```

The agent appears in the inventory when the issue is resolved.
