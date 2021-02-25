# Hello World

## Installation
### Flux2
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


`helm install { AppName } chart`