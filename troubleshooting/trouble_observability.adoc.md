# Troubleshooting observability

After you install the observability component, the component might be stuck and an `Installing` status is displayed. 

## Symptom: MultiClusterObservability resource status stuck

If the observability status is stuck in an `Installing` status after you install and create the Observability custom resource definition (CRD), it is possible that there is no value defined for the `spec:storageConfig:storageClass` parameter. Alternatively, the observability component automatically finds the default `storageClass`, but if there is no value for the storage, the component remains stuck with the `Installing` status.

## Resolving the problem: MultiClusterObservability resource status stuck

If you have this problem, complete the following steps:

1. Verify that the observability components are installed:
   1. To verify that the `multicluster-observability-operator`, run the following command:

      ```
      kubectl get pods -n open-cluster-management|grep observability
      ```
   2. To verify that the appropriate CRDs are present, run the following command: 

      ```
      kubectl get crd|grep observ
      ```

      The following CRDs must be displayed before you enable the component:

      ```
      multiclusterobservabilities.observability.open-cluster-management.io   
      observabilityaddons.observability.open-cluster-management.io          
      observatoria.core.observatorium.io
      ```
2. If you create your own storageClass for a Bare Metal cluster, see [Persistent storage using NFS](https://docs.redhat.com/documentation/en-us/openshift_container_platform/4.12/html-single/storage/index#persistent-storage-using-nfs).
3. To ensure that the observability component can find the default storageClass, update the `storageClass` parameter in the `multicluster-observability-operator` custom resource definition. Your parameter might resemble the following value:

```
storageclass.kubernetes.io/is-default-class: "true"
```

The observability component status is updated to a _Ready_ status when the installation is complete. If the installation fails to complete, the _Fail_ status is displayed.
