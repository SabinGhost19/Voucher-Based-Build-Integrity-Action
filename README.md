# VBBI Voucher Attestor

Custom GitHub Action pentru Voucher-Based Build Integrity.

Actiunea construieste un voucher VBBI pe baza unei liste ordonate de pasi, aplica HMAC chaining, calculeaza radacina Merkle si ataseaza rezultatul ca atestare Cosign.

## Inputs

- `image`: referinta OCI imutabila in format digest
- `commit-sha`: commit SHA asociat build-ului
- `step-spec-path`: fisier JSON cu pasi in ordinea corecta
- `hmac-key`: cheia HMAC folosita in modul implicit `shared-secret`
- `vault-address`: URL-ul Vault pentru modul `vault-transit`
- `vault-token`: token Vault folosit de action pentru Transit
- `vault-namespace`: namespace Vault optional
- `vault-transit-mount`: mount path-ul Transit
- `vault-transit-key`: cheia Transit folosita pentru lantul HMAC
- `vault-transit-algorithm`: algoritmul HMAC Transit, implicit `sha2-256`
- `slsa-level`: nivelul SLSA declarat in voucher
- `attestation-type`: tipul de atestare Cosign

## Format `step-spec-path`

```json
[
  { "name": "linting", "file": "/tmp/vbbi/lint-results.json" },
  { "name": "unit-tests", "file": "/tmp/vbbi/test-results.json" },
  { "name": "image-build", "file": "/tmp/vbbi/build-results.json" },
  { "name": "vulnerability-scan", "file": "/tmp/vbbi/scan-results.json" }
]
```

## Exemplu de folosire

```yaml
- name: Attach VBBI attestation
  uses: SabinGhost19/vbbi-voucher-attestor@v1
  with:
    image: ghcr.io/${{ github.repository_owner }}/demo-api@${{ steps.build.outputs.digest }}
    commit-sha: ${{ github.sha }}
    step-spec-path: /tmp/vbbi/step-spec.json
    hmac-key: ${{ secrets.VBBI_HMAC_KEY }}
    slsa-level: 3
```

## Exemplu cu Vault Transit

```yaml
- name: Attach VBBI attestation via Vault Transit
  uses: SabinGhost19/vbbi-voucher-attestor@v1
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