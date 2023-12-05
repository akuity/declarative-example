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
  https://akuity.cloud/<name>/argocd/declarative-reference
  ```

### Connect the clusters
Provision on agent for the cluster named `kind`.
```
akuity argocd cluster create --instance-name=declarative-reference kind
```

Apply the agent install manifests to the cluster.
```
akuity argocd cluster get-agent-manifests --instance-name=declarative-reference kind | kubectl apply -f -
```
- Check the progress of the agent installation with `k get pods -n akuity`.

### Log into the `argocd` CLI
```
argocd login \
    $(akuity argocd instance get -o json | jq -r '.[] | select(.name == "declarative-reference") | .id').cd.akuity.cloud \
    --username admin \
    --password akuity-argocd \
    --grpc-web
```