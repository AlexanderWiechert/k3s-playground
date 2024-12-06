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