# flux2-kind

```bash
$ flux install -v v0.41.2 --watch-all-namespaces=false --export | kubectl apply -f -
$ kubectl apply -f config/bootstrap/tenant/flux-system/gotk-sync.yaml
```