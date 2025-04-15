### Build image
```bash
docker build -t sazzadr/argocd-image-updater:0.1 .
```

### Push image
```bash
docker push sazzadr/argocd-image-updater:0.1
```

### Start minikube ###
```bash
minikube start
minikube addons enable ingress
```

### Install argocd ###
```bash
kubectl create namespace argocd

helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

helm install argocd argo/argo-cd -n argocd -f k8s/argocd/argocd-values.yaml
```

Get the argocd admin password.

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

Access the argocd UI from browser. With `admin` username and the password from output.

```bash
minikube service argocd-server -n argocd
```

Install the root-app in ArgoCD

```bash
kubectl apply -f k8s/argocd/root-app.yaml
```

### Add ingress ip to hosts ####
```bash
kubectl get ingress -n demo-app --watch

# Output
NAME               CLASS   HOSTS            ADDRESS        PORTS   AGE
demo-app-ingress   nginx   demo-app.local                  80      4s
demo-app-ingress   nginx   demo-app.local   192.168.49.2   80      22s
```

Add `192.168.49.2    demo-app.local` in the `/etc/hosts` file.

### Access the application ###
Open browser and go to `http://demo-app.local`


### Install argocd-image-updater ###

[//]: # (```bash)

[//]: # (helm install argocd-image-updater argo/argocd-image-updater -n argocd -f k8s/argocd/image-updater.yaml)

[//]: # (```)


```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml


kubectl edit application demo-app -n argocd

annotations:
  argocd-image-updater.argoproj.io/image-list: sazzadr/argocd-image-updater
  argocd-image-updater.argoproj.io/argocd-image-updater.update-strategy: latest/newest-build
  argocd-image-updater.argoproj.io/write-back-method: git:secret:argocd/git-creds


kubectl -n argocd create secret generic git-creds \
  --from-file=sshPrivateKey=/home/sazzad/.ssh/id_rsa
```
