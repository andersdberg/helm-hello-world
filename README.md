# Hello World

## Flux2
### Installation
Install CLI on MacOS
`brew install fluxcd/tap/flux`

Check cluster prerequisites
`flux check --pre`

Bootstrap
```
flux bootstrap github \
  --components=source-controller,kustomize-controller,helm-controller,notification-controller \
  --components-extra=image-reflector-controller,image-automation-controller \
  --path=clusters/development \
  --version=latest \
  --owner=andersdberg \
  --repository=helm-hello-world \
  --private=false \
  --personal=true
```

Verify
`flux check`

#### Terraform
Flux can also be bootstrapped with the flux Terraform provider that offers two resources: `flux_install` and `flux_sync`

### Manage Helm Releases
#### Define Chart Source
To use a Git repository, create a `GitRepository` resource. Can also use `HelmRepository`.
```
cat <<EOF | kubectl apply -f -
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: GitRepository
metadata:
  name: app
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/andersdberg/helm-hello-world
  ref:
    branch: main
  ignore: |
    # exclude all
    /*
    # include charts directory
    !/charts/
EOF
```

#### Define Helm Release
```
cat <<EOF | kubectl apply -f -
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: app
  namespace: default
spec:
  interval: 5m
  chart:
    spec:
      chart: app
      version: '0.1.0'
      sourceRef:
        kind: GitRepository
        name: app
        namespace: flux-system
      interval: 1m
  values:
    replicaCount: 2
EOF
```