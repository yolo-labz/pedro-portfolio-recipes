# sigstore-attestation-verify

Sign every release artifact with a GitHub-native SLSA L2 build provenance
attestation. Users verify with one `gh` command — no cosign install, no trust
roots to configure.

## Problem

Asking users to install `cosign` and `slsa-verifier` just to check a download's
provenance is friction that no one does. `gh attestation verify` ships with the
GitHub CLI they already have.

## Snippet

Release workflow step (in a job with `id-token: write` and
`attestations: write`):

```yaml
- name: Attest build provenance
  uses: actions/attest-build-provenance@v2
  with:
    subject-path: 'dist/*'

- name: Attest SBOM
  uses: actions/attest-sbom@v2
  with:
    subject-path: 'dist/*'
    sbom-path: 'sbom.cdx.json'
```

User verification (after `gh release download vX.Y.Z -D dist/`):

```bash
# Verify build provenance for every downloaded artifact.
find dist -type f -exec gh attestation verify {} \
  --repo yolo-labz/my-plugin \;

# One-liner, any file:
gh attestation verify ./my-plugin-v1.2.3.tar.gz --repo yolo-labz/my-plugin
```

## Why

`actions/attest-build-provenance@v2` generates SLSA L2 provenance using
Sigstore's public good transparency log — no key management, no secrets in the
workflow. GitHub's API serves the attestation, so the user doesn't need
`slsa-verifier`, `cosign`, or the Sigstore trust root. Verification is a
single command any developer with `gh` can run.

## When NOT to use

Don't ship an attestation-gated download flow when your users are non-technical
end-users who install via Homebrew or an installer — the formula or installer
should verify on their behalf, not surface `gh attestation verify` in the UX.
Also don't try to replace SLSA L3 with this for supply-chain-critical binaries
where a formal claim matters; for L3, `slsa-framework/slsa-github-generator`
remains the reference path.

## Reference

- https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations
