# Akuity Platform Declarative Example
A complete example of using the `akuity` CLI to declaratively manage Argo CD instances on the Akuity Platform.

This repository is accompanied by a [complete tutorial](https://docs.akuity.io/tutorials/declarative-akuity-platform) in the Akuity Platform documentation.

## Steps
### Log into the `akuity` CLI.
```
akuity login
```
- Open the link displayed, enter the code.

## Set your organization name in the `akuity` config.
```
akuity config set --organization-name=<name>
```
- Replace `<name>` with your organization name.

### Create the Argo CD instance on the Akuity Platform.
```
akuity argocd apply -f akuity-platform/example
```

### Connect the clusters
Apply the agent install manifests to the cluster.
```
akuity argocd cluster get-agent-manifests --instance-name=example kind | kubectl apply -f -
```

### Log into the `argocd` CLI
```
argocd login \
    $(akuity argocd instance get example -o json | jq -r '.id').cd.akuity.cloud \
    --username admin \
    --password akuity-argocd \
    --grpc-web
```

### List the Applications
```
argocd app list
```

Example output:
```
NAME                   CLUSTER     NAMESPACE       PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                               PATH                    TARGET
argocd/bootstrap       in-cluster  argocd          default  Synced  Healthy  Auto-Prune  <none>      https://github.com/akuity/declarative-example      apps/                   HEAD
argocd/helm-guestbook  kind        helm-guestbook  default  Synced  Healthy  Auto-Prune  <none>      https://github.com/morey-tech/argocd-example-apps  general/helm-guestbook  HEAD
argocd/sync-waves      kind        sync-waves      default  Synced  Healthy  Auto-Prune  <none>      https://github.com/morey-tech/argocd-example-apps  general/sync-waves      HEAD
```
