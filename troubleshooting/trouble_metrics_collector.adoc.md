# Troubleshooting metrics-collector

When the `observability-client-ca-certificate` secret is not refreshed in the managed cluster, you might receive an internal server error.

## Symptom: metrics-collector cannot verify observability-client-ca-certificate

There might be a managed cluster, where the metrics are unavailable. If this is the case, you might receive the following error from the `metrics-collector` deployment: 

```
error: response status code is 500 Internal Server Error, response body is x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "observability-client-ca-certificate")
```

## Resolving the problem: metrics-collector cannot verify observability-client-ca-certificate

If you have this problem, complete the following steps:

1. Log in to your managed cluster. 
2. Delete the secret named, `observability-controller-open-cluster-management.io-observability-signer-client-cert` that is in the `open-cluster-management-addon-observability` namespace. Run the following command:

   ```
   oc delete secret observability-controller-open-cluster-management.io-observability-signer-client-cert -n open-cluster-management-addon-observability
   ```

   **Note:** The `observability-controller-open-cluster-management.io-observability-signer-client-cert` is automatically recreated with new certificates. 

The `metrics-collector` deployment is recreated and the `observability-controller-open-cluster-management.io-observability-signer-client-cert` secret is updated.
