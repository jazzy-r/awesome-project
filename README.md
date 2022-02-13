# awesome-project


- https://fluxcd.io/docs/get-started/
- https://k9scli.io/
- https://kind.sigs.k8s.io/docs/user/quick-start/


## Install Kind 

```
brew install kind
```

## Install Flux 

```
brew install fluxcd/tap/flux
```

## Install K9S 

```
brew install derailed/k9s/k9s
```

## Fork this repository

- https://github.com/jazzy-r/awesome-project.git

# Set GitHub Tokens #

```
export GITHUB_TOKEN=<your-personal-token>

export GITHUB_USER=<your-github-username>
```

# Create Local Kubernetes Cluster #

```
kind create cluster

kubectl cluster-info --context kind-kind

```

# Check that Flux is happy #

```
flux check --pre
```

# Creates Namespaces # 

```
kubectl create namespace production
kubectl create namespace staging
```

# Github Booststrap # 

- repository flag is the name of the repository that will be created in your Github account
- path flag is folder in which flux will put the files it needs

```
flux bootstrap github \
 --owner jazzy-r \
 --repository dev-talk-demo \
 --branch main \
 --path . \
 --personal

```

# Clone the repository flux created to your local environment

```
git clone git@github.com:<your-github-username>/dev-talk-demo.git
```

# Create a Flux git source for staging branch

flux create source git awesome-project-staging \
    --url=https://github.com/jazzy-r/awesome-project.git \
    --branch=staging \
    --interval=30s \
    --export > ./awesome-project-staging-source.yaml

# Create a Flux git source for production branch

 flux create source git awesome-project-production \
    --url=https://github.com/jazzy-r/awesome-project.git \
    --branch=main \
    --interval=30s \
    --export > ./awesome-project-production-source.yaml


# Commit these two files

# Check the status in Flux

'''
flux get all
'''

### Check gitrepositories in K9s

```
flux create kustomization awesome-project-production \
    --target-namespace=production \
    --source=awesome-project-production \
    --path="./helm" \
    --prune=true \
    --interval=1m \
    --export > ./production-kustomization.yaml
```
```
flux create kustomization awesome-project-staging \
    --target-namespace=staging \
    --source=awesome-project-staging \
    --path="./helm" \
    --prune=true \
    --interval=1m \
    --export > ./staging-kustomization.yaml
```
### Commit these two files

### Check Kustomizations in K9s

# Update the staging Kustomizations

```
---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: staging-helm
  namespace: flux-system
spec:
  interval: 1m0s
  path: ./helm
  prune: true
  sourceRef:
    kind: GitRepository
    name: awesome-project-staging
  targetNamespace: staging

---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: awesome-project-staging
  namespace: flux-system
spec:
  interval: 1m0s
  path: ./releases/staging
  prune: true
  sourceRef:
    kind: GitRepository
    name: awesome-project-staging
  targetNamespace: staging
  dependsOn:
    - name: staging-helm
```