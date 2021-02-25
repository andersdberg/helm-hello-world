# Hello World

## Minikube
Start with a fresh minikube instance prior to each demo, `minikube delete && minikube start`

## Flux2
### Installation
#### Install CLI on MacOS
`brew install fluxcd/tap/flux`

#### Check cluster prerequisites
`flux check --pre`

#### Bootstrap
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

#### Verify
`flux check`

#### Terraform Bootstrap Alternative
Flux can also be bootstrapped with the flux Terraform provider that offers two resources: `flux_install` and `flux_sync`

### Operation
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
EOF
```

#### Define Helm Release
```
cat <<EOF | kubectl apply -f -
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: app
  namespace: flux-system
spec:
  interval: 5m
  chart:
    spec:
      chart: charts/app
      version: '0.1.0'
      sourceRef:
        kind: GitRepository
        name: app
        namespace: flux-system
      interval: 1m
EOF
```

## ArgoCD
### Installation
#### Create `argocd` namespace
`kubectl create namespace argocd`

#### Install ArgoCD
`kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`

#### Install ArgoCD CLI (MacOS)
`brew install argocd`

#### Expose API server (and dashboard)
`kubectl port-forward svc/argocd-server -n argocd 8080:443`
The username is `admin` and the password can be retrieved with:
`kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2`

#### Login to argocd cli with the username and password from above
`argocd login localhost:8080`

#### Add the cluster
`argocd cluster add minikube`

### Operation
#### Create the application
```
cat <<EOF | kubectl apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/andersdberg/helm-hello-world
    targetRevision: HEAD
    path: charts/app
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
EOF
```