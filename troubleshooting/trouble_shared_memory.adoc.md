# Troubleshooting PostgreSQL shared memory error

If you have a large environment, you might encounter a PostgreSQL shared memory error that impacts search results and the topology view for applications.
 
## Symptom: PostgreSQL shared memory error

An error message resembling the following appears in the `search-api` logs: `ERROR: could not resize shared memory segment "/PostgreSQL.1083654800" to 25031264 bytes: No space left on device (SQLSTATE 53100)`

## Resolving the problem: PostgreSQL shared memory error

To resolve the issue, update the PostgreSQL resources found in the `search-postgres` ConfigMap. Complete the following steps to update the resources:

1. Run the following command to switch to the `open-cluster-management` project:

+
```bash
oc project open-cluster-management
```

1. Increase the `search-postgres` pod memory. The following command increases the memory to `16Gi`:

+
```bash
oc patch search -n open-cluster-management search-v2-operator --type json -p '[{"op": "add", "path": "/spec/deployments/database/resources", "value": {"limits": {"memory": "16Gi"}, "requests": {"memory": "32Mi", "cpu": "25m"}}}]'
```

1. Run the following command to prevent the search operator from overwriting your changes:

+
```bash
oc annotate search search-v2-operator search-pause=true
```

1. Run the following command to update the resources in the `search-postgres` YAML file:

+
```bash
oc edit cm search-postgres -n open-cluster-management
```
+
See the following example for increasing resources:

+
```yaml
  postgresql.conf: |-
    work_mem = '128MB' # Higher values allocate more memory
    max_parallel_workers_per_gather = '0' # Disables parallel queries
    shared_buffers = '1GB' # Higher values allocate more memory
```
+
Make sure to save your changes before exiting.

1. Run the following command to restart the `postgres` and `api` pod.

+
```bash
oc delete pod search-postgres-xyz search-api-xzy
```

1. To verify your changes, open the `search-postgres` YAML file and confirm that the changes you made to `postgresql.conf:` are present by running the following command:

+
```bash
oc get cm search-postgres -n open-cluster-management -o yaml
```

See [Search customization and configurations](../console/search_console.adoc#search-customization) for more information on adding environment variables.
