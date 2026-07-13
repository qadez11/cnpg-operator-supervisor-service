# CNPG Supervisor Service

Carvel bundle for VMware vSphere Supervisor Service with:

- CloudNativePG operator `1.30.0`
- CloudNativePG Barman Cloud plugin `0.13.0`
- Barman Cloud sidecar image packed through `imgpkg` image lock

Package refName: `cnpg.fling.vsphere.vmware.com`

Bundle image:

```bash
harbor.itnovel.ru/vsphere/cnpg-supervisor-service:1.30.0-barman-0.13.0
```

## Files

```text
cnpg-supervisor-service/
  .imgpkg/images.yml
  config/
    operator/
      cnpg-1.30.0.yaml
      plugin-barman-cloud-0.13.0.yaml
    overlay/
      images.yaml
      kbld-config.yaml
      namespace.yaml
      user-rbac.yaml
      webhooks.yaml
    values.yaml

cnpg-vsphere-1.30.0-def.yaml
```

## Values

Only one package value is exposed:

```yaml
namespace: svc-cnpg
```

Images are not package values. They are locked and relocated by Carvel through:

```text
cnpg-supervisor-service/.imgpkg/images.yml
```

Locked images:

- `ghcr.io/cloudnative-pg/cloudnative-pg:1.30.0`
- `ghcr.io/cloudnative-pg/plugin-barman-cloud:v0.13.0`
- `ghcr.io/cloudnative-pg/plugin-barman-cloud-sidecar:v0.13.0`

## Render Check

```bash
ytt -f cnpg-supervisor-service/config/
```

Render with locked images:

```bash
ytt -f cnpg-supervisor-service/config/ | kbld -f - -f cnpg-supervisor-service/.imgpkg/images.yml
```

Check that the old namespace is gone:

```bash
ytt -f cnpg-supervisor-service/config/ | rg cnpg-system
```

The command should return no output.

## Push Bundle

```bash
imgpkg push \
  -b harbor.itnovel.ru/vsphere/cnpg-supervisor-service:1.30.0-barman-0.13.0 \
  -f cnpg-supervisor-service/ \
  --lock-output cnpg-supervisor-service/bundle.lock.yml
```

This pushes the bundle metadata. To copy the bundle together with all locked images into Harbor, run:

```bash
imgpkg copy \
  -b harbor.itnovel.ru/vsphere/cnpg-supervisor-service:1.30.0-barman-0.13.0 \
  --to-repo harbor.itnovel.ru/vsphere/cnpg-supervisor-service-relocated \
  --lock-output cnpg-supervisor-service/bundle.lock.yml
```

## Package Definition

Apply or publish:

```bash
cnpg-vsphere-1.30.0-def.yaml
```

It defines:

- `Package`: `cnpg.fling.vsphere.vmware.com.1.30.0+0.13.0`
- `PackageMetadata`: `cnpg.fling.vsphere.vmware.com`
- `appplatform.vmware.com/requires-ha-supervisor: "true"`
- template chain: `ytt`, `kbld`, `kapp`

## User Access

End-user access is handled through aggregate Kubernetes roles:

- `edit-cnpg` aggregates into the standard `edit` role
- `view-cnpg` aggregates into the standard `view` role

Users and groups should be granted access to Tanzu/vSphere Namespaces through normal vSphere Namespace permissions. The bundle does not bind SSO users directly.
