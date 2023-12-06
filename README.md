# Akuity Platform Declarative Example
A complete example of using the `akuity` CLI to declaratively manage Argo CD instances on the Akuity Platform.

## Manually Creating an Instance
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
- View the instance in the UI at (replacing `<name>` with your organization name):
  ```
  https://akuity.cloud/<name>/argocd/example
  ```

This will create an Argo CD instance based the definition in the `argocd.yaml` manifest, and the configurations in the `argocd-cm.yaml` and `argocd-secret.yaml` (which follow the same format as the open-source).

The `cluster.yaml` manifest contains the configuration for provisioning an agent that will connect a Kubernetes cluster to the Argo CD instance. In the next step, you'll install the agent into the cluster.

The `Application` in the `bootstrap-app.yaml` manifest will also be deployed. It points to a folder containing other `Applications` and `ApplicationSets`, therefore automatically bootstrapping your Argo CD instance.

The `Applications` deployed by the `bootstrap` `Application` may appear in an `Unknown` sync status due to the unavilable destination cluster at the time of creation. After completing the next step, they will eventually retry (up to 5 minutes) and sync successfully. You can speed this up by clicking refresh after the agent has become healthy on the cluster.

### Connect the clusters
Apply the agent install manifests to the cluster.
```
akuity argocd cluster get-agent-manifests --instance-name=example kind | kubectl apply -f -
```
- Check the progress of the agent installation with `kubectl get pods -n akuity`.

### Log into the `argocd` CLI
```
argocd login \
    $(akuity argocd instance get example -o json | jq -r '.id').cd.akuity.cloud \
    --username admin \
    --password akuity-argocd \
    --grpc-web
```

The command on the second line is a bit complex, so let's break it down:
- `akuity argocd instance get example -o json` will retrieve the Argo CD instance named `example` from your organization on the Akuity Platform and output the metadata in JSON.
- The output is then piped to `jq -r '.id'`. We use `jq` here to filter the JSON output down to the `.id` attribute for the `example` Argo CD instance.
- This is all encapsulated in `$(...)` which allows us to use the id as the sub-domain of the FQDN `<id>.cd.akuity.cloud` for the server URL in the `argocd login` command. 

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

### Managing only Applications
The `akuity argocd apply -f` command can apply a folder containing only `Applications`, `ApplicationSets`, and `AppProjects` without including an Argo CD definition by specifying the instance name with `--name`.
```
akuity argocd apply -f apps/ --name example
```