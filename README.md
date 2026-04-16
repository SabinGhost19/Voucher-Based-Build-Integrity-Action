# VBBI Voucher Attestor

Custom GitHub Action for Voucher-Based Build Integrity.

This action builds a VBBI voucher from an ordered list of build steps, applies HMAC chaining, computes the Merkle root, and attaches the result as a Cosign attestation.

## Inputs

- `image`: immutable OCI image reference in digest form
- `commit-sha`: commit SHA associated with the build
- `step-spec-path`: JSON file containing the ordered build steps
- `hmac-key`: HMAC key used in the default `shared-secret` mode
- `vault-address`: Vault URL used for `vault-transit` mode
- `vault-token`: Vault token used by the action for Transit
- `vault-namespace`: optional Vault namespace
- `vault-transit-mount`: Transit mount path
- `vault-transit-key`: Transit key used for the HMAC chain
- `vault-transit-algorithm`: Transit HMAC algorithm, default is `sha2-256`
- `slsa-level`: SLSA level declared in the voucher
- `attestation-type`: Cosign attestation type

## Format `step-spec-path`

```json
[
  { "name": "linting", "file": "/tmp/vbbi/lint-results.json" },
  { "name": "unit-tests", "file": "/tmp/vbbi/test-results.json" },
  { "name": "image-build", "file": "/tmp/vbbi/build-results.json" },
  { "name": "vulnerability-scan", "file": "/tmp/vbbi/scan-results.json" }
]
```

## Usage Example

```yaml
- name: Attach VBBI attestation
  uses: SabinGhost19/Voucher-Based-Build-Integrity-Action@v1
  with:
    image: ghcr.io/${{ github.repository_owner }}/demo-api@${{ steps.build.outputs.digest }}
    commit-sha: ${{ github.sha }}
    step-spec-path: /tmp/vbbi/step-spec.json
    hmac-key: ${{ secrets.VBBI_HMAC_KEY }}
    slsa-level: 3
```

## Vault Transit Example

```yaml
- name: Attach VBBI attestation via Vault Transit
  uses: SabinGhost19/Voucher-Based-Build-Integrity-Action@v1
  with:
    image: ghcr.io/${{ github.repository_owner }}/demo-api@${{ steps.build.outputs.digest }}
    commit-sha: ${{ github.sha }}
    step-spec-path: /tmp/vbbi/step-spec.json
    vault-address: ${{ secrets.VAULT_ADDR }}
    vault-token: ${{ secrets.VAULT_TOKEN }}
    vault-transit-key: vbbi-hmac
    vault-transit-algorithm: sha2-256
    slsa-level: 3
```