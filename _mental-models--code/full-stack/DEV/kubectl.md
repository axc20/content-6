# kubectl

```shell
# See all the clusters you can connect to
kubectl config get-contexts

# The name change is permanent and stored in your kubeconfig file (~/.kube/config)
kubectl config rename-context {old context name} {new context name}

# Set the current context (modify context settings)
kubectl config set-context {context name}

# Switch to use a context (more common, best practice)
kubectl config use-context {context name}

# Verify current context
kubectl config current-context
```
