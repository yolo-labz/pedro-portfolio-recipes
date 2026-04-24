# pedro-portfolio-recipes

Short, copy-pasteable patterns I actually use — Claude Code plugins, RAG retrieval,
NixOS, multi-cloud Terraform, and supply-chain hardening. Each recipe is one
directory with a problem statement, a working snippet, a tradeoff note, and a
reference.

Not a tutorial site. Not LeetCode. These are fragments lifted from the stacks I
ship in [yolo-labz](https://github.com/yolo-labz) plugins and the writeups on my
[blog](https://blog.home301server.com.br/).

## Recipes

| Topic | Stack |
|-------|-------|
| [claude-code-plugin-skill-skeleton](recipes/claude-code-plugin-skill-skeleton/) | Claude Code plugin · SKILL.md + run.sh |
| [chrome-multi-profile-bash](recipes/chrome-multi-profile-bash/) | macOS · bash 3.2 · Chrome Local State catalog |
| [rag-pgvector-hybrid-search](recipes/rag-pgvector-hybrid-search/) | Postgres 16 · pgvector · BM25 hybrid |
| [sigstore-attestation-verify](recipes/sigstore-attestation-verify/) | GitHub Actions · `gh attestation verify` · SLSA L2 |
| [nixos-flake-overlay-pattern](recipes/nixos-flake-overlay-pattern/) | Nix flakes · overlays · input precedence |
| [terraform-azure-gcp-parallel-modules](recipes/terraform-azure-gcp-parallel-modules/) | Terraform · Azure + GCP · module-pair |
| [sops-nix-age-secrets](recipes/sops-nix-age-secrets/) | sops-nix · age · NixOS secrets |
| [lefthook-go-rust-typescript](recipes/lefthook-go-rust-typescript/) | lefthook · commitlint · polyglot pre-commit |

## Layout

```
recipes/<topic>/
  README.md     # problem, snippet, tradeoff, anti-pattern, reference
```

Each recipe stands alone. No build step. No shared framework.

## Evidence

Related public work:

- Claude Code plugins: [wa](https://github.com/yolo-labz/wa) ·
  [claude-mac-chrome](https://github.com/yolo-labz/claude-mac-chrome) ·
  [kokoro-speakd](https://github.com/yolo-labz/kokoro-speakd) ·
  [claude-classroom-submit](https://github.com/yolo-labz/claude-classroom-submit) ·
  [linkedin-chrome-copilot](https://github.com/yolo-labz/linkedin-chrome-copilot) ·
  [fand](https://github.com/yolo-labz/fand)
- Long-form writeups: [blog.home301server.com.br](https://blog.home301server.com.br/)

## Contributing

Solo-maintained. Issues welcome; PRs accepted for typo/link fixes.

## License

MIT — see [LICENSE](LICENSE).
