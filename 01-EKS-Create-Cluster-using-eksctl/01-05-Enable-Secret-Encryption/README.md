# Enabling secret encryption on an existing cluster
- If you enable secrets encryption, the Kubernetes secrets are encrypted using the AWS KMS key that you select. 
- The KMS key must meet the following conditions:
    - Symmetric
    - Can encrypt and decrypt data
    - Created in the same AWS Region as the cluster
    - If the KMS key was created in a different account, the IAM principal must have access to the KMS key.

## Enable Encryption
````
# To automatically re-encrypt your secrets

eksctl utils enable-secrets-encryption \
    --cluster my-cluster \
    --key-arn arn:aws:kms:region-code:account:key/key

# To opt-out of automatically re-encrypting your secrets

eksctl utils enable-secrets-encryption 
    --cluster my-cluster \
    --key-arn arn:aws:kms:region-code:account:key/key \
    --encrypt-existing-secrets=false
````

## KMS key protection
- If you enable secrets encryption for an existing cluster and the KMS key that you use is ever deleted, then there's no way to recover the cluster. 
- If you delete the KMS key, you permanently put the cluster in a degraded state.