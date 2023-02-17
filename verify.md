### Verify if key was used

_replace image with your image, and to avoid warnings use digest instead of tag_

```bash
cosign verify --key hashivault://ci-system --rekor-url https://rekor.apps.open-svc-sts.k1wl.p1.openshiftapps.com quay.io/sallyom/pipelines-vote-api:latest
cosign verify-attestation --key hashivault://ci-system --rekor-url https://rekor.apps.open-svc-sts.k1wl.p1.openshiftapps.com --type spdxjson --attachment-tag-prefix sbom- quay.io/sallyom/pipelines-vote-api:latest
```

### Verify with keyless workflow

```bash
cosign verify quay.io/sallyom/pipelines-vote-api:latest --certificate-oidc-issuer https://rh-oidc.s3.us-east-1.amazonaws.com/21o4ml1f5kchd6ee9nmh5dhc6efqba38 --rekor-url https://rekor.apps.open-svc-sts.k1wl.p1.openshiftapps.com --certificate-identity https://kubernetes.io/namespaces/test-sigstore/serviceaccounts/pipeline
cosign verify-attestation --type spdxjson --attachment-tag-prefix sbom- quay.io/sallyom/pipelines-vote-api:latest --certificate-oidc-issuer https://rh-oidc.s3.us-east-1.amazonaws.com/21o4ml1f5kchd6ee9nmh5dhc6efqba38 --rekor-url https://rekor.apps.open-svc-sts.k1wl.p1.openshiftapps.com --certificate-identity https://kubernetes.io/namespaces/test-sigstore/serviceaccounts/pipeline
cosign verify-attestation --type vuln --attachment-tag-prefix sarif- quay.io/sallyom/pipelines-vote-api:latest --certificate-oidc-issuer https://rh-oidc.s3.us-east-1.amazonaws.com/21o4ml1f5kchd6ee9nmh5dhc6efqba38 --rekor-url https://rekor.apps.open-svc-sts.k1wl.p1.openshiftapps.com --certificate-identity https://kubernetes.io/namespaces/test-sigstore/serviceaccounts/pipeline
```
