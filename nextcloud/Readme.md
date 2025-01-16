# install ingress controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

# add external dns 
kubectl apply -k kustomize/base/

# ssl support
kubectl create namespace cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.1/cert-manager.yaml

# install all
kubectl apply -f nextcloud

# cleanup
kubectl delete -f nextcloud


# certificate
kubectl apply -f nextcloud/cert

# route53
ich verwende route53 um die txt records für dei dns validierung automatisch zu erstellen-
kubectl create secret generic route53-credentials-secret \
--from-literal=accessKeyID=<YOUR_ACCESS_KEY_ID> \
--from-literal=secretAccessKey=<YOUR_SECRET_ACCESS_KEY> \
-n cert-manager
