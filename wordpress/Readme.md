# install ingress controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

# ssl support
kubectl create namespace cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.1/cert-manager.yaml

# install all
kubectl apply -f wordpress
kubectl apply -f wordpress/ert


# cleanup
kubectl delete -f wordpress

#debug mysql mariadb-client-core-10.5