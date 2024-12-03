# Troubleshooting OpenShift monitoring service

Observability service in a managed cluster needs to scrape metrics from the OpenShift Container Platform monitoring stack. The `metrics-collector` is not installed if the OpenShift Container Platform monitoring stack is not ready.

## Symptom: OpenShift monitoring service is not ready

The `endpoint-observability-operator-x` pod checks if the `prometheus-k8s` service is available in the `openshift-monitoring` namespace. If the service is not present in the `openshift-monitoring` namespace, then the `metrics-collector` is not deployed. You might receive the following error message: `Failed to get prometheus resource`.

## Resolving the problem: OpenShift monitoring service is not ready

If you have this problem, complete the following steps:

1. Log in to your OpenShift Container Platform cluster.
2. Access the `openshift-monitoring` namespace to verify that the `prometheus-k8s` service is available.
3. Restart `endpoint-observability-operator-x` pod in the `open-cluster-management-addon-observability` namespace  of the managed cluster.
