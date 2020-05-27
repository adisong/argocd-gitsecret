# argocd-test

# Install argocd

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

# Patch argocd to add support for git secret

```bash
kubens argocd
# Generate GPG key and add it to this repo using git secret tell
# gpg --armor --export-secret-key $GPG_ID > $GPG_FILE_PATH
kubectl create secret generic argocd-gitsecret-gpg --from-file=$GPG_FILE_PATH
kubectl patch configmap argocd-cm -p "$(cat ./argocd-tools/cm-patch_gitsecret-raw.yaml)"
kubectl patch deployment argocd-repo-server -p "$(cat ./argocd-tools/repo-server-patch_gitsecret-raw.yaml)"
```

# Add Argo APP that uses git secrets

```bash
kubectl create ns guestbook
argocd app create gitsecrets-raw-guestbook \
    --config-management-plugin gitsecrets-raw \
    --repo https://github.com/adisong/argocd-test \
    --path guestbook-gitsecret \
    --dest-server https://kubernetes.default.svc \
    --dest-namespace guestbook \
    --revision master \
    --sync-policy none
```