# Troubleshooting Klusterlet with degraded conditions

The Klusterlet degraded conditions can help to diagnose the status of Klusterlet agents on managed cluster. If a Klusterlet is in the degraded condition, the Klusterlet agents on managed cluster might have errors that need to be troubleshooted. See the following information for Klusterlet degraded conditions that are set to `True`.

## Symptom: Klusterlet is in the degraded condition

After deploying a Klusterlet on managed cluster, the `KlusterletRegistrationDegraded` or `KlusterletWorkDegraded`
condition displays a status of _True_.

## Identifying the problem: Klusterlet is in the degraded condition

1. Run the following command on the managed cluster to view the Klusterlet status:

   ```
   kubectl get klusterlets klusterlet -oyaml
   ```
2. Check `KlusterletRegistrationDegraded` or `KlusterletWorkDegraded` to see if the condition is set to `True`. Proceed to _Resolving the problem_ for any degraded conditions that are listed.

## Resolving the problem: Klusterlet is in the degraded condition

See the following list of degraded statuses and how you can attempt to resolve those issues:

* If the `KlusterletRegistrationDegraded` condition with a status of _True_ and the condition reason is: _BootStrapSecretMissing_,
you need create a bootstrap secret on `open-cluster-management-agent` namespace.
* If the `KlusterletRegistrationDegraded` condition displays _True_ and the condition reason is a _BootstrapSecretError_,
or _BootstrapSecretUnauthorized_, then the current bootstrap secret is invalid. Delete the current bootstrap secret and recreate a valid bootstrap
secret on `open-cluster-management-agent` namespace.
* If the `KlusterletRegistrationDegraded` and `KlusterletWorkDegraded` displays _True_ and the condition reason is
_HubKubeConfigSecretMissing_, delete the Klusterlet and recreate it.
* If the `KlusterletRegistrationDegraded` and `KlusterletWorkDegraded` displays _True_ and the condition reason is:
_ClusterNameMissing_, _KubeConfigMissing_, _HubConfigSecretError_, or _HubConfigSecretUnauthorized_, delete the hub cluster kubeconfig
secret from `open-cluster-management-agent` namespace. The registration agent will bootstrap again to get a new hub cluster kubeconfig secret.
* If the `KlusterletRegistrationDegraded` displays _True_ and the condition reason is _GetRegistrationDeploymentFailed_,
or _UnavailableRegistrationPod_, you can check the condition message to get the problem details and attempt to resolve.
* If the `KlusterletWorkDegraded` displays _True_ and the condition reason is _GetWorkDeploymentFailed_ ,or _UnavailableWorkPod_,
you can check the condition message to get the problem details and attempt to resolve.
