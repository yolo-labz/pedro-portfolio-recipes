# terraform-azure-gcp-parallel-modules

Mirror-image Terraform modules for Azure and GCP so the consumer root picks
the cloud at `locals.cloud` and the rest of the config reads identically.

## Problem

Cross-cloud Terraform usually devolves into two parallel root configs that
drift, or into a single root with `count = var.cloud == "azure" ? 1 : 0`
littered across every resource. Both are unmaintainable.

## Snippet

Directory layout:

```
modules/
  network/
    azure/  { main.tf, outputs.tf, variables.tf }
    gcp/    { main.tf, outputs.tf, variables.tf }
  compute/
    azure/  { ... }
    gcp/    { ... }
root/
  main.tf
  terraform.tfvars
```

`root/main.tf`:

```hcl
locals {
  cloud = "azure"   # flip to "gcp" — nothing else changes.
}

module "network" {
  source   = "../modules/network/${local.cloud}"
  vpc_cidr = "10.42.0.0/16"
  tags     = { project = "demo" }
}

module "compute" {
  source     = "../modules/compute/${local.cloud}"
  subnet_id  = module.network.subnet_id
  image      = "ubuntu-24.04"
  node_count = 3
}
```

The two `network` modules expose the SAME output names (`subnet_id`, `vpc_id`)
with identical variable shapes (`vpc_cidr`, `tags`). The per-cloud provider
config lives inside each module, so `root/main.tf` stays provider-neutral.

## Why

A thin abstraction at the module boundary forces the provider-specific code
into one place per cloud and leaves the consumer free of conditionals.
`source = "../modules/x/${local.cloud}"` is a compile-time decision — not the
`count = ? : 0` pattern that still plans both branches.

## When NOT to use

Don't mirror modules when the two clouds diverge on a primitive the abstraction
can't paper over (e.g. Azure Storage immutability policies have no GCS parallel
for WORM retention). Leak the difference explicitly — a separate module per
cloud with DIFFERENT names beats a lying uniform interface. Also skip this
pattern for single-cloud-plus-one-exception; `count = 1` is fine when the
exception is truly one resource.

## Reference

- https://developer.hashicorp.com/terraform/language/modules/develop/structure
