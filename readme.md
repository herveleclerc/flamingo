# Demo flamingo


## For OCI use


```bash

export CR_PAT="ghp_...."
echo $CR_PAT | docker login ghcr.io -u USERNAME --password-stdin

```

# Prepare Flamingo deployment

```bash

export FSA_VERSION=v2.6.6-fl.4-main-0d5eae51
kustomize build "https://github.com/flux-subsystem-argo/flamingo//release?ref=${FSA_VERSION}"  > manifests/${FSA_VERSION}.yaml

```


## package oci - release

```bash
flux push artifact oci://ghcr.io/herveleclerc/manifests/argocd:1.0.0 --path=. --source=$(git config --get remote.origin.url) --revision="$(git branch --show-current)@sha1:$(git rev-parse HEAD)"

```


## Create OCIRepository and deploy  manifests


```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: OCIRepository
metadata:
  name: fsa-demo
  namespace: flux-system
spec:
  interval: 30s
  url: oci://ghcr.io/herveleclerc/manifests/argocd
  ref:
    tag: 1.0.0

---

apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: flamingo-demo
  namespace: flux-system
spec:
  prune: true
  interval: 2m
  path: "."
  sourceRef:
    kind: OCIRepository
    name: flamingo-demo
  timeout: 3m
---

```


