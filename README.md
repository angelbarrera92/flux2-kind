# Flux 2 upgrade from v0.41.2 to v2.0.1

In a local cluster with a two phase setup.

## Pre-requisites

- Clone this repository.
- Install flux [v0.41.2](https://github.com/fluxcd/flux2/releases/tag/v0.41.2) in your path as `flux` *`/usr/local/bin/flux`*
- Install flux [v2.0.1](https://github.com/fluxcd/flux2/releases/tag/v2.0.1) in your path as `flux2` *`/usr/local/bin/flux2`*
- An empty Kubernetes cluster. Tested with `colima` on `1.24.3+k3s1`.

## Install the starting point

```bash
$ flux install -v v0.41.2 --watch-all-namespaces=false --export | kubectl apply -f -
$ kubectl apply -f config/bootstrap/tenant/flux-system/gotk-sync.yaml
```

At this point you should have everything in healthy status:

```bash
$ kubectl get kustomization,gitrepository -n flux-system
NAME                                                        AGE     READY   STATUS
kustomization.kustomize.toolkit.fluxcd.io/init-flux2-kind   9m31s   True    Applied revision: main@sha1:dc8973285633c9221062458c69d03d6faf8e8936
kustomization.kustomize.toolkit.fluxcd.io/flux2-kind        9m31s   True    Applied revision: main@sha1:dc8973285633c9221062458c69d03d6faf8e8936

NAME                                                URL                                                AGE     READY   STATUS
gitrepository.source.toolkit.fluxcd.io/flux2-kind   https://github.com/angelbarrera92/flux2-kind.git   9m31s   True    stored artifact for revision 'main@sha1:dc8973285633c9221062458c69d03d6faf8e8936'
```

## The flux2 branch

I've created a flux2 branch starting from the main branch. This branch contains the flux2 manifests generated by running:

```bash
$ flux2 install -v v2.0.1 --watch-all-namespaces=false --export > config/bootstrap/tenant/flux-system/gotk-components.yaml
```

It also contains a modification in the `config/bootstrap/tenant/flux-system/gotk-sync.yaml` file to use this new branch in the `GitRepository` resource.

*Check the [Final words](#final-words) section for more information about the final fix.*

### Update to flux v2

I was updating flux with flux itself. So im simulating here a merge in main of the flux2 branch by moving to the flux2 branch and running:

```bash
$ git checkout flux2
$ kubectl apply -f config/bootstrap/tenant/flux-system/gotk-sync.yaml
```

### Final words

There could be some race conditions while deploying the new flux version if you do not force the flux components first by setting [config/bootstrap/tenant/flux-system/kustomization.yaml](config/bootstrap/tenant/flux-system/kustomization.yaml):

```yaml
patches:
# Every flux resource must be deployed with the label odp/phase=init to be deployed by the init-co kustomization
- target:
    labelSelector: app.kubernetes.io/part-of=flux
  patch: |-
    - op: add
      path: /metadata/labels/flux2-kind~1phase
      value: init
```
