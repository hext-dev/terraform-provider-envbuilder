# terraform-provider-envbuilder

Fork of coder/terraform-provider-envbuilder using hext-dev/envbuilder for cache hash computation.

## When to Update

**Only update this provider when hext-dev/envbuilder changes affect cache hash computation.** The provider uses envbuilder's code to compute what the cached image digest *should* be. If that logic changes in envbuilder, the provider needs the new version or cache lookups will fail.

Runtime-only envbuilder changes (e.g., container startup fixes) don't require a provider update.

## Links

- **Registry**: https://registry.terraform.io/providers/hext-dev/envbuilder
- **Envbuilder fork**: https://github.com/hext-dev/envbuilder

## Current Setup

Using dev_overrides to redirect `coder/envbuilder` → local binary:

```
# /home/coder/.terraformrc
provider_installation {
  dev_overrides {
    "coder/envbuilder" = "/home/coder/.terraform.d/plugins"
  }
  direct {}
}
```

Template still uses `source = "coder/envbuilder"` which gets overridden.

**To switch to registry instead:** Remove dev_overrides, change template source to `hext-dev/envbuilder`, run `terraform init -upgrade`.

## Release

```bash
# Update go.mod replace directive, then:
go mod tidy && git commit -am "bump envbuilder" && git push
git tag v1.0.X && git push origin v1.0.X  # GitHub Actions handles release

# Update local binary
cd /tmp && gh release download v1.0.X -R hext-dev/terraform-provider-envbuilder -p '*linux_amd64*'
unzip -o terraform-provider-envbuilder_*.zip
sudo cp terraform-provider-envbuilder_v1.0.X /home/coder/.terraform.d/plugins/
```

## Debug

```bash
coder state pull <workspace> | jq '.resources[] | select(.type == "envbuilder_cached_image") | {exists, image}'
```
