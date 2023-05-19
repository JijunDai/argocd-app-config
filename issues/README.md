## argocd-repo-server can not start issue

**Describe the bug**

Logs:
```
time="2022-12-10T18:22:34Z" level=info msg="Generating self-signed TLS certificate for this session"
time="2022-12-10T18:22:35Z" level=info msg="Initializing GnuPG keyring at /app/config/gpg/keys"
time="2022-12-10T18:22:35Z" level=info msg="gpg --no-permission-warning --logger-fd 1 --batch --gen-key /tmp/gpg-key-recipe3098385539" dir= execID=71abc
time="2022-12-10T18:22:41Z" level=error msg="`gpg --no-permission-warning --logger-fd 1 --batch --gen-key /tmp/gpg-key-recipe3098385539` failed exit status 2" execID=71abc
time="2022-12-10T18:22:41Z" level=info msg=Trace args="[gpg --no-permission-warning --logger-fd 1 --batch --gen-key /tmp/gpg-key-recipe3098385539]" dir= operation_name="exec gpg" time_ms=6009.092806
time="2022-12-10T18:22:41Z" level=fatal msg="`gpg --no-permission-warning --logger-fd 1 --batch --gen-key /tmp/gpg-key-recipe3098385539` failed exit status 2"
```
solving this by removing:
```yaml
seccompProfile:
  type: RuntimeDefault
```
from repo-server `containerSecurityContext`

## Access ArgoCD UI over Nginx Ingress-Controller
Ordinarily , this would have been easy to configure by setting up an ingress service for the dashboard but it is very hard to configure for Argo CD due to the self SSL termination. You have to face the issue with the SSL redirects of the service.

Here is a solution that not deployed the ingress with SSL. So if you’re looking for a solution with SSL then this is probably not the right place.

- First edit the argocd-server deployment to add the insecure flag to enable http connections . You need to make the change as shown in the below image.



```yaml
containers:
- command:
  - argocd-server
  - --insecure

```

- Next make the change in nginx ingress controller deployment to add the enable-ssl-passthrough flag

```yaml
 spec:
       containers:
       - args:
       # ....
         - --enable-ssl-passthrough  
```

- Argocd ingress 

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    ingress.kubernetes.io/proxy-body-size: 100M
    ingress.kubernetes.io/app-root: "/"
    kubernetes.io/ingress.class: "nginx"
    alb.ingress.kubernetes.io/ssl-passthrough: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "false"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
  name: argocd-ui-ingress
  namespace: argocd
spec:
  ingressClassName: nginx
  rules:
  - host: p1-argocdui.mgcorp.co
    http:
      paths:
      - backend:
          service:
            name: argocd-server
            port:
              number: 80
        path: /
        pathType: Prefix
```

## Commands

```bash
# install ArgoCD in k8s
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# access ArgoCD UI
kubectl get svc -n argocd
kubectl port-forward svc/argocd-server 8080:443 -n argocd

# login with admin user and below token (as in documentation):
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo
# you can change and delete init password
```
</br>

#### Links

* Install ArgoCD: [https://argo-cd.readthedocs.io/en/stable/getting_started/#1-install-argo-cd](https://argo-cd.readthedocs.io/en/stable/getting_started/#1-install-argo-cd)

* Login to ArgoCD: [https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli](https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli)

* ArgoCD Configuration: [https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/)

