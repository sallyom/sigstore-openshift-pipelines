_replace image with your image, and to avoid warnings use digest instead of tag_

### Verify sigstore signed artifacts

Adjust to match your cluster

```bash
export IMAGE=quay.io/sallyom/pipelines-vote-api:latest
export BASE=apps.open-svc-sts.k1wl.p1.openshiftapps.com
export SERVICEACCOUNT=pipeline
export NAMESPACE=test-sigstore
export OIDC_ISSUER_URL=https://rh-oidc.s3.us-east-1.amazonaws.com/21o4ml1f5kchd6ee9nmh5dhc6efqba38
```

#### Verify with self-managed key in signing secret & copied from tekton-chains ns to test-sigstore ns

```bash
# verify signature
cosign verify --key k8s://test-sigstore/signing-secrets \
    --rekor-url https://$BASE \
    $IMAGE

# verify SBOM attestation
cosign verify-attestation --key k8s://test-sigstore/signing-secrets \
    --rekor-url https://rekor.$BASE \
    --type spdxjson --attachment-tag-prefix sbom- \
    $IMAGE

# verify vulnerability report attestatation
cosign verify-attestation --type https://cosign.sigstore.dev/attestation/vuln/v1 \
    --key k8s://test-sigstore/signing-secrets \
    --attachment-tag-prefix sarif- \
    --rekor-url https://rekor.$BASE \
    $IMAGE
```

### Verify with keyless workflow

```bash
# verify signature
cosign verify $IMAGE \
    --certificate-oidc-issuer $OIDC_ISSUER_URL \
    --rekor-url https://rekor.$BASE \
    --certificate-identity https://kubernetes.io/namespaces/$NAMESPACE/serviceaccounts/$SERVICEACCOUNT

# verify SBOM attestation
cosign verify-attestation --type spdxjson \
    --attachment-tag-prefix sbom- \
    --certificate-oidc-issuer $OIDC_ISSUER_URL \
    --rekor-url https://rekor.$BASE \
    --certificate-identity https://kubernetes.io/namespaces/$NAMESPACE/serviceaccounts/$SERVICEACCOUNT \
    $IMAGE

# verify vulnerability report attestation
cosign verify-attestation --type https://cosign.sigstore.dev/attestation/vuln/v1 \
    --attachment-tag-prefix sarif- \
    --certificate-oidc-issuer $OIDC_ISSUER_URL \
    --rekor-url https://rekor.$BASE \
    --certificate-identity https://kubernetes.io/namespaces/$NAMESPACE/serviceaccounts/$SERVICEACCOUNT \
    $IMAGE
```

#### Verify with KMS key in vault

```bash
# verify signature
cosign verify \
    --key hashivault://ci-system \
    --rekor-url https://rekor.$BASE \
    $IMAGE

# verify SBOM attestation
cosign verify-attestation \
    --key hashivault://ci-system \
    --rekor-url https://rekor.$BASE \
    --type spdxjson --attachment-tag-prefix sbom- \
    $IMAGE

# verify vulnerability report attestatation
cosign verify-attestation \
    --type https://cosign.sigstore.dev/attestation/vuln/v1 \
    --key hashivault://ci-system \
    --attachment-tag-prefix sarif- \
    --rekor-url https://rekor.$BASE \
    $IMAGE
```

#### Extract predicate from attestation

This example extracts the SBOM from attestation

```bash
# with self-managed key
cosign verify-attestation \
    --key k8s://test-sigstore/signing-secrets \
    --attachment-tag-prefix sbom- \
    --type spdxjson \
    --rekor-url https://rekor.$BASE \
    $IMAGE | jq '.payload | @base64d | fromjson | .predicate.Data'

# with keyless
cosign verify-attestation \
    --type spdxjson \
    --attachment-tag-prefix sbom- \
    --certificate-oidc-issuer $OIDC_ISSUER_URL \
    --rekor-url https://rekor.$BASE \
    --certificate-identity https://kubernetes.io/namespaces/$NAMESPACE/serviceaccounts/$SERVICEACCOUNT \
    $IMAGE | jq '.payload | @base64d | fromjson | .predicate.Data'
```
