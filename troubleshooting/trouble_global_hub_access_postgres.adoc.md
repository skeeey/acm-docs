# Troubleshooting by accessing the PostgreSQL database 

## Symptom: Errors with multicluster global hub

You might experience various errors with multicluster global hub. You can access the provisioned PostgreSQL database to view messages that might be helpful for troubleshooting issues with multicluster global hub.

## Resolving the problem: Accessing the PostgresSQL database 

    There are two ways to access the provisioned PostgreSQL database.

* Using the `ClusterIP` service

  ```
  oc exec -it multicluster-global-hub-postgres-0 -c multicluster-global-hub-postgres -n multicluster-global-hub -- psql -U postgres -d hoh

  # Or access the database installed by crunchy operator
  oc exec -it $(kubectl get pods -n multicluster-global-hub -l postgres-operator.crunchydata.com/role=master -o jsonpath='{.items..metadata.name}') -c database -n multicluster-global-hub -- psql -U postgres -d hoh -c "SELECT 1"
  ```
* `LoadBalancer`
+ 
  * Expose the service type to `LoadBalancer` provisioned by default:

    ```
    cat <<EOF | oc apply -f -
    apiVersion: v1
    kind: Service
    metadata:
      name: multicluster-global-hub-postgres-lb
      namespace: multicluster-global-hub
    spec:
      ports:
      - name: postgres
        port: 5432
        protocol: TCP
        targetPort: 5432
      selector:
        name: multicluster-global-hub-postgres
      type: LoadBalancer
    EOF
    ```

    Run the following command to get your the credentials:

    ```
    # Host
    oc get svc postgres-ha -ojsonpath='{.status.loadBalancer.ingress[0].hostname}'

    # Password
    oc get secrets -n multicluster-global-hub postgres-pguser-postgres -o go-template='{{index (.data) "password" | base64decode}}'
    ```
    + 
  * Expose the service type to `LoadBalancer` provisioned by crunchy operator:

    ```
    oc patch postgrescluster postgres -n multicluster-global-hub -p '{"spec":{"service":{"type":"LoadBalancer"}}}'  --type merge
    ```

    Run the following command to get your the credentials:

    ```
    # Host
    oc get svc -n multicluster-global-hub postgres-ha -ojsonpath='{.status.loadBalancer.ingress[0].hostname}'

    # Username
    oc get secrets -n multicluster-global-hub postgres-pguser-postgres -o go-template='{{index (.data) "user" | base64decode}}'

    # Password
    oc get secrets -n multicluster-global-hub postgres-pguser-postgres -o go-template='{{index (.data) "password" | base64decode}}'

    # Database
    oc get secrets -n multicluster-global-hub postgres-pguser-postgres -o go-template='{{index (.data) "dbname" | base64decode}}'
    ```
