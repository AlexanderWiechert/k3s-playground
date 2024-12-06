# k3s-playground

## standard installation
curl -sfL https://get.k3s.io | sh -

## disable traefik
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --disable traefik" sh

### cleanup
/usr/local/bin/k3s-uninstall.sh

cp /etc/rancher/k3s/k3s.yaml ~/.kube/config

# test installation
kubectl get all --all-namespaces

# install ingress controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml


# test loadbalancer 
kubectl apply -f test/nginx-http.yaml

curl http://nc.sbb.org
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

# enable HTTPS

# install cert-manager
kubectl create namespace cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.1/cert-manager.yaml
kubectl get pods -n cert-manager
NAME                                       READY   STATUS              RESTARTS   AGE
cert-manager-67f7b4f67d-qvftr              0/1     ContainerCreating   0          4s
cert-manager-cainjector-564b978556-wfmnz   0/1     ContainerCreating   0          4s
cert-manager-webhook-7d54ddf58b-fzcrh      0/1     ContainerCreating   0          4s

# apply issuer
kubectl apply -f cert/clusterissuer.yaml -n default
clusterissuer.cert-manager.io/letsencrypt-prod created

# test issuer
kubectl get clusterissuers
NAME               READY   AGE
letsencrypt-prod   True    77s

# apply certificate
kubectl apply -f cert/certificate.yaml -n default
certificate.cert-manager.io/letsencrypt-prod-cert created

# test certificate
kubectl describe certificate  letsencrypt-prod-cert -n default


# create new https resources 

kubectl apply -f test/nginx-https.yaml
ingress.networking.k8s.io/nginx-https created
