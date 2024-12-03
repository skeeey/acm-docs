# Troubleshooting multiline YAML parsing

When you want to use the `fromSecret` function to add contents of a `Secret` resource into a `Route` resource, the contents are displayed incorrectly. 

## Symptom: Troubleshooting multiline YAML parsing

When the managed cluster and hub cluster are the same cluster the certificate data is redacted, so the contents are not parsed as a template JSON string. You might receive the following error messages:

```json
message: >-
            [spec.tls.caCertificate: Invalid value: "redacted ca certificate
            data": failed to parse CA certificate: data does not contain any
            valid RSA or ECDSA certificates, spec.tls.certificate: Invalid
            value: "redacted certificate data": data does not contain any valid 
            RSA or ECDSA certificates, spec.tls.key: Invalid value: "": no key specified]   
```

## Resolving the problem: Troubleshooting multiline YAML parsing

Configure your certificate policy to retrieve the hub cluster and managed cluster `fromSecret` values. Use the `autoindent` function to update your certificate policy with the following content:

```yaml
                 tls:
                    certificate: |
                      {{ print "{{hub fromSecret "open-cluster-management" "minio-cert" "tls.crt" hub}}" | base64dec | autoindent }}
```
