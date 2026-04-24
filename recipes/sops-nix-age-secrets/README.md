# sops-nix-age-secrets

Encrypt NixOS secrets at rest with `age` keys derived from the host's SSH
ed25519 host key. Commit the encrypted blob; let the host decrypt at activation.

## Problem

Secrets can't land in the Nix store (world-readable). But secrets MUST land in
the config repo for the build to be reproducible. `sops-nix` resolves that by
decrypting at activation time into a tmpfs path the service reads.

## Snippet

Derive an age recipient from the host's SSH key and add it to
`.sops.yaml`:

```bash
ssh-to-age -i /etc/ssh/ssh_host_ed25519_key.pub
# age1abcdef...  ← paste this into .sops.yaml as a creation rule
```

`.sops.yaml`:

```yaml
creation_rules:
  - path_regex: hosts/macbook-pro/secrets\.yaml$
    key_groups:
      - age:
          - age1abcdef...   # macbook-pro host key
          - age1user...     # Pedro's user key (for editing)
```

`hosts/macbook-pro/configuration.nix`:

```nix
{ config, inputs, ... }: {
  imports = [ inputs.sops-nix.nixosModules.sops ];

  sops = {
    defaultSopsFile = ./secrets.yaml;
    age.sshKeyPaths = [ "/etc/ssh/ssh_host_ed25519_key" ];
    secrets."postgres/password" = {
      owner = "postgres";
      mode  = "0400";
    };
  };

  services.postgresql.authentication = lib.mkForce ''
    local all postgres peer
  '';
  systemd.services.myapp.environment.PGPASSWORD_FILE =
    config.sops.secrets."postgres/password".path;
}
```

Edit: `sops hosts/macbook-pro/secrets.yaml`. The file stays encrypted on disk
and in git; the decrypted value only lives at
`/run/secrets/postgres/password` on the live host.

## Why

Binding encryption to the host SSH key means provisioning a new host is a
matter of adding one `age1…` recipient to `.sops.yaml` — no secret
re-distribution, no shared passphrase. `sops-nix` activation runs before any
service that needs the secret, so the read path is well-defined.

## When NOT to use

Don't use for secrets that rotate frequently (API tokens with <24h lifetime),
or for secrets a process fetches on boot from a secret manager (Vault, AWS
Secrets Manager). `sops-nix` is for stable-ish secrets the host owns.

## Reference

- https://github.com/Mic92/sops-nix
