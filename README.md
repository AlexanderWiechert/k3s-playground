# k3s-playground

## standard installation
curl -sfL https://get.k3s.io | sh -

## disable traefik
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --disable traefik" sh

## with vpn 
curl -sfL https://get.k3s.io |INSTALL_K3S_EXEC="server --disable traefik"  sh -s - server --node-ip=192.168.6.1 --bind-address=192.168.6.1 --advertise-address=192.168.6.1 --tls-san=192.168.6.1

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


# use wireguard connection 
sudo curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --disable traefik --flannel-iface=wg0" sh

sudo curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server="https://192.168.6.1:6443"--token=K10e0dc69ba6b9d2c987e6158d3ad77da26d2381ec29d8611f6faceec70944c3228::server:9e62e8c943957b830e1717a16d72480a --flannel-iface=wg0"


# use external iface
export k3s_token="K1012afbe6650bba384f32ed756c66a7ffc980f6fc741ecd3dbecabc2471d5d350c::server:5a9325e68f1ff55abbe76eaa23e38baf"
export k3s_url="https://152.53.46.232:6443"
curl -sfL https://get.k3s.io | K3S_URL=${k3s_url} K3S_TOKEN=${k3s_token} sh -


ExecStart=/usr/local/bin/k3s \
agent --server=https://192.168.6.1:6443 --token=K10e0dc69ba6b9d2c987e6158d3ad77da26d2381ec29d8611f6faceec70944c3228::server:9e62e8c943957b830e1717a16d72480a --flannel-iface=wg0\