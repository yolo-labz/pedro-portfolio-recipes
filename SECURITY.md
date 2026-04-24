# Security Policy

## Scope

This repo ships copy-pasteable code recipes. There is no binary, no service, no
runtime. The "security surface" is the recipe text itself — if a snippet here
would introduce a vulnerability when applied verbatim, that is a bug worth
reporting.

## Reporting a vulnerability

**Do not file public GitHub issues for security concerns.** Use GitHub's private
vulnerability reporting:

1. Go to https://github.com/yolo-labz/pedro-portfolio-recipes/security/advisories/new
2. Describe the recipe (path), the concern, and a suggested fix
3. Submit

PGP is not supported. GitHub PVR handles encryption and access control.

## Response

Solo-maintained. Acknowledgement within 7 days; fix timing depends on severity
and triage queue.

## Out of scope

- Upstream vulnerabilities in the tools a recipe references (Postgres, Nix,
  Terraform, etc.) — report those to the upstream project.
- Recipes clearly marked "anti-pattern" or "when NOT to use" — those document
  behavior to avoid.
