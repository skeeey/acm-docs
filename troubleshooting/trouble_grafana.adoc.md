# Troubleshooting Grafana

When you query some time-consuming metrics in the Grafana explorer, you might encounter a `Gateway Time-out` error.

## Symptom: Grafana explorer gateway timeout

If you hit the `Gateway Time-out` error when you query some time-consuming metrics in the Grafana explorer, it is possible that the timeout is caused by the Grafana in the `open-cluster-management-observability` namespace.

## Resolving the problem: Configure the Grafana

If you have this problem, complete the following steps:

1. Verify that the default configuration of Grafana has expected timeout settings:
   1. To verify that the default timeout setting of Grafana, run the following command:

      ```
      oc get secret grafana-config -n open-cluster-management-observability -o jsonpath="{.data.grafana\.ini}" | base64 -d | grep dataproxy -A 4
      ```

      The following timeout settings should be displayed:

      ```
      [dataproxy]
      timeout = 300
      dial_timeout = 30
      keep_alive_seconds = 300
      ```
   2. To verify the default data source query timeout for Grafana, run the following command: 

      ```
      oc get secret/grafana-datasources -n open-cluster-management-observability -o jsonpath="{.data.datasources\.yaml}" | base64 -d | grep queryTimeout
      ```

      The following timeout settings should be displayed:

      ```
      queryTimeout: 300s
      ```
2. If you verified the default configuration of Grafana has expected timeout settings, then you can configure the Grafana in the `open-cluster-management-observability` namespace by running the following command:

   ```
   oc annotate route grafana -n open-cluster-management-observability --overwrite haproxy.router.openshift.io/timeout=300s
   ```

Refresh the Grafana page and try to query the metrics again. The `Gateway Time-out` error is no longer displayed.
