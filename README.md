# k3s-playground

curl -sfL https://get.k3s.io | sh -

cp /etc/rancher/k3s/k3s.yaml ~/.kube/config

kubectl get all --all-namespaces

kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-native.yaml

kubectl apply -f metallb/ip-addresspool.yaml
