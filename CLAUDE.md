# terraform-provider-envbuilder

Fork of coder/terraform-provider-envbuilder using hext-dev/envbuilder for cache hash computation.

## When to Update

**Only update this provider when hext-dev/envbuilder changes affect cache hash computation.** The provider uses envbuilder's code to compute what the cached image digest *should* be. If that logic changes in envbuilder, the provider needs the new version or cache lookups will fail.

Runtime-only envbuilder changes (e.g., container startup fixes) don't require a provider update.

## Links

- **Registry**: https://registry.terraform.io/providers/hext-dev/envbuilder
- **Envbuilder fork**: https://github.com/hext-dev/envbuilder

## Current Setup

Template uses `source = "hext-dev/envbuilder"` directly from the Terraform Registry.

No dev_overrides needed - `/home/coder/.terraformrc` just has `provider_installation { direct {} }`.

## Release

```bash
# Update go.mod replace directive, then:
go mod tidy && git commit -am "bump envbuilder" && git push
git tag v1.0.X && git push origin v1.0.X  # GitHub Actions handles release
```

GPG key: `C1469A531E975F1C` (terraform@hext.dev)

## Debug

```bash
coder state pull <workspace> | jq '.resources[] | select(.type == "envbuilder_cached_image") | {exists, image}'
```
