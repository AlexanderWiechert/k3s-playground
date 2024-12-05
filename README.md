# k3s-playground

## standard installation
curl -sfL https://get.k3s.io | sh -

#disable traefik
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --disable traefik" sh


cp /etc/rancher/k3s/k3s.yaml ~/.kube/config

# test installation
kubectl get all --all-namespaces

#install meltallb
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-native.yaml

# test metllb installtion
kubectl get pods -n metallb-system
NAME                          READY   STATUS    RESTARTS   AGE
controller-6dd967fdc7-6g2jq   1/1     Running   0          5m48s
speaker-7lqt2                 1/1     Running   0          5m48s

# apply adress pool
kubectl apply -f metallb/IPAddressPool.yaml
ipaddresspool.metallb.io/demo-pool created

# test adress pool
kubectl get IPAddressPool -A
NAMESPACE        NAME         AUTO ASSIGN   AVOID BUGGY IPS   ADDRESSES
metallb-system   first-pool   true          false             ["152.53.46.232/32"]

# apply layer 2 config
kubectl apply -f metallb/L2Advertisement.yaml
l2advertisement.metallb.io/demo created

# test layer 2 config
kubectl apply -f metallb/L2Advertisement.yaml
l2advertisement.metallb.io/demo created

kubectl get L2Advertisement -A
NAMESPACE        NAME   IPADDRESSPOOLS   IPADDRESSPOOL SELECTORS   INTERFACES
metallb-system   demo   ["demo-pool"]


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
Name:         letsencrypt-prod-cert
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  cert-manager.io/v1
Kind:         Certificate
Metadata:
Creation Timestamp:  2024-12-05T10:33:29Z
Generation:          1
Resource Version:    1110
UID:                 67e2a03e-649e-4c43-a979-32b8dd642873
Spec:
Dns Names:
nc.sbb.org
Issuer Ref:
Kind:       ClusterIssuer
Name:       letsencrypt-prod
Secret Name:  letsencrypt-prod-secret
Status:
Conditions:
Last Transition Time:        2024-12-05T10:33:29Z
Message:                     Issuing certificate as Secret does not exist
Observed Generation:         1
Reason:                      DoesNotExist
Status:                      True
Type:                        Issuing
Last Transition Time:        2024-12-05T10:33:29Z
Message:                     Issuing certificate as Secret does not exist
Observed Generation:         1
Reason:                      DoesNotExist
Status:                      False
Type:                        Ready
Next Private Key Secret Name:  letsencrypt-prod-cert-tjwpx
Events:
Type    Reason     Age   From                                       Message
  ----    ------     ----  ----                                       -------
Normal  Issuing    2m1s  cert-manager-certificates-trigger          Issuing certificate as Secret does not exist
Normal  Generated  2m1s  cert-manager-certificates-key-manager      Stored new private key in temporary Secret resource "letsencrypt-prod-cert-tjwpx"
Normal  Requested  2m1s  cert-manager-certificates-request-manager  Created new CertificateRequest resource "letsencrypt-prod-cert-1"

# create new https ingress 
kubectl create namespace nginx-ingress
namespace/nginx-ingress created

kubectl apply -f test/nginx-https.yaml -n nginx-ingress
ingress.networking.k8s.io/nginx-https created


# test
kubectl get ingress -A
NAMESPACE   NAME                        CLASS    HOSTS        ADDRESS   PORTS     AGE
default     cm-acme-http-solver-7kwlx   <none>   nc.sbb.org             80        102m
default     nginx-https                 <none>   nc.sbb.org             80, 443   88m


kubectl describe ingress nginx-https
Name:             nginx-https
Labels:           <none>
Namespace:        default
Address:
Ingress Class:    <none>
Default backend:  <default>
TLS:
letsencrypt-prod-secret terminates nc.sbb.org
Rules:
Host        Path  Backends
  ----        ----  --------
nc.sbb.org
/   nginx-service:80 (10.42.0.7:80)
Annotations:  cert-manager.io/cluster-issuer: letsencrypt-prod
kubernetes.io/ingress.class: nginx-ingress
nginx.ingress.kubernetes.io/force-ssl-redirect: true
nginx.ingress.kubernetes.io/rewrite-target: /
spec.ingressClassName: nginx
Events:
Type    Reason             Age    From                       Message
  ----    ------             ----   ----                       -------
Normal  CreateCertificate  5m42s  cert-manager-ingress-shim  Successfully created Certificate "letsencrypt-prod-secret"