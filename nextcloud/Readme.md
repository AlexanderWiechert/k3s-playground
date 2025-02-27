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


# troubleshooting
> Einige fehlende optionale Indizes wurden erkannt. Gelegentlich werden neue Indizes hinzugefügt (von Nextcloud oder installierten Anwendungen), um die Datenbankleistung zu verbessern. Das Hinzufügen von Indizes kann manchmal eine Weile dauern und die Leistung vorübergehend beeinträchtigen, daher wird dies bei Upgrades nicht automatisch durchgeführt. Sobald die Indizes hinzugefügt wurden, sollten Abfragen an diese Tabellen schneller sein. Verwende den Befehl `occ db:add-missing-indices`, um sie hinzuzufügen.Fehlende Indizes: "systag_by_objectid" in Tabelle "systemtag_object_mapping". Weitere Informationen findest du in der Dokumentation ↗.

kctl exec -it nextcloud-5c5c885956-zkslk -- su -s /bin/sh -c 'php occ db:add-missing-indices' www-data

>One or more mimetype migrations are available. Occasionally new mimetypes are added to better handle certain file types. Migrating the mimetypes take a long time on larger instances so this is not done automatically during upgrades. Use the command `occ maintenance:repair --include-expensive` to perform the migrations.

kctl exec -it nextcloud-5c5c885956-zkslk -- su -s /bin/sh -c 'php occ maintenance:repair --include-expensive' www-data