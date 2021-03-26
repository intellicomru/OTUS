#### Настраиваем окружение для работы с GKE : ####

# 1. Обновим Debian  
apt-get -y update  
apt-get -y upgrade  

https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/  

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"   

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl   

echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list   

sudo apt-get install apt-transport-https ca-certificates gnupg   

curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring   /usr/share/keyrings/cloud.google.gpg add -  

sudo apt-get update && sudo apt-get install google-cloud-sdk  

gcloud init   
выбрать вариант - новый аккаунт. пройти с логином через браузер. там получить ключ для работы из виртуалки    


 gcloud beta container --project "tensile-analyst-305614" clusters create "muzclaster" --zone "europe-central2-a" --no-enable-basic-auth --cluster-version "1.17.17-gke.1101" --release-channel "None" --machine-type "e2-medium" --image-type "COS" --disk-type "pd-standard" --disk-size "100" --metadata disable-legacy-endpoints=true --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --num-nodes "3" --enable-stackdriver-kubernetes --enable-ip-alias --network "projects/tensile-analyst-305614/global/networks/default" --subnetwork "projects/tensile-analyst-305614/regions/europe-central2/subnetworks/default" --default-max-pods-per-node "110" --no-enable-master-authorized-networks --addons HorizontalPodAutoscaling,HttpLoadBalancing --enable-autoupgrade --enable-autorepair --max-surge-upgrade 1 --max-unavailable-upgrade 0 --node-locations "europe-central2-a"  



gcloud container clusters describe muzclaster --zone=europe-central2-a  --format="value(endpoint)"   
  
34.116.201.49  
     
gcloud container clusters describe muzclaster  --zone=europe-central2-a      --format="value(masterAuth.clusterCaCertificate)"     
    
 mkdir /keys   
 
**/keys/kubeconfig.yaml**    
```
apiVersion: v1
kind: Config
clusters:
- name: muzclaster
  cluster:
    server: https://34.116.201.49
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURLekNDQWhPZ0F3SUJBZ0lSQVBDbTVFRURBaVBYcDNtbURrYmpqZlF3RFFZSktvWklodmNOQVFFTEJRQXcKTHpFdE1Dc0dBMVVFQXhNa01UUXhaalZoWlRRdE1qZzBaaTAwTkdRMExUZzRZbUV0TldVd1ltTmtPREEwWWpKagpNQjRYRFRJeE1ETXlOVEU1TWpRME5Wb1hEVEkyTURNeU5ESXdNalEwTlZvd0x6RXRNQ3NHQTFVRUF4TWtNVFF4ClpqVmhaVFF0TWpnMFppMDBOR1EwTFRnNFltRXROV1V3WW1Oa09EQTBZakpqTUlJQklqQU5CZ2txaGtpRzl3MEIKQVFFRkFBT0NBUThBTUlJQkNnS0NBUUVBbmY4TUV4bmJFcDRINDIveFhMejBFUjBoMERUZzB2Nks3UWRlUEk4WQo1T2FCWlZWcWRQcmJzTVU4alRpZFMwUGluYnlRaFRtUVpvVG9xazJsTEFxV0I5R2JhNTQ3VzZhMU9lcmR4SzJ6CmdUTkVVa0Rqc0ZwaWVERmVOM1VkWGtxeUxFSWNxbU0ycHNHcHVvNlZZSUV2OWhMQmZuS3NXVUFwV1V6SVRTUnYKOS8wZWdkZVdDMVNmNjIvbythYmNNSld0SjJQRzZ6MGduYTdYRWRzc2kyMHpXTCtRUzNPOEFGb1FIamZZdC9hRgpVL3dHK2pRYzhrZzdYek9lNndPMEhXSG1ja2ZtaVFqM2t1VDdodVN1Tnpka09Hb1FmbWhUdE1rZWJ6UGFMT3U1ClVYRTlMODJ4QXc4Ny9mcmhsV0k2Z0JlNjVGb3lVY0JoRWFGSWM2TXI3WDZGWHdJREFRQUJvMEl3UURBT0JnTlYKSFE4QkFmOEVCQU1DQWdRd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZEJnTlZIUTRFRmdRVWIwaXU5MTRDQ3U4RQpUQTlhazZHTU5mOWd3all3RFFZSktvWklodmNOQVFFTEJRQURnZ0VCQUl3UTRGRmZvVGlqMHdkQm1MTUFpaGNwCnR5Y095YXF6MmlwaGJoNmIweExMN3U2UnI1UDBLMU1zaThmbXcxRmcxRHROTmR2TjN6eXZGd2lUbWh4WGthUDEKd2JEUytpSEVyWVYvN29QNUEwK3N4aGpmNk5kZ0R1NDhHZXFHZGxkbno4enRGRDBkaU1HVHNrTEMyWDRlcU82egpHR0ZacU44OTJHS0lQLzJRVDhkRnd0NTNJeUNHdjRCK1FPampFbUNybkNzaVlkN2hIRlNWb04wRXBoalpmMzhJCnhrdlBiUEpQVzljL3JCTTJwcWJhQVgxaW14TWdYWlRQd0JucE5zdGJvejV3RlhpR29obkJjalNnQ3U2a1NRK3oKU2lPbmtLSnlWZE9PQjFHUkIrL2dBUXNGVjVySmV3TzBrYnc3WHk3aG94OUFBdHNhNGhETEhibXg5NDJLalJZPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
users:
- name: ci-cd-pipeline-gsa
  user:
    auth-provider:
      name: gcp
contexts:
- context:
    cluster: muzclaster
    user: ci-cd-pipeline-gsa
  name: muzclaster-ci-cd
current-context: muzclaster-ci-cd  
```

export KUBECONFIG=/keys/kubeconfig.yaml    

cd /keys  
gcloud iam service-accounts keys create gsa-key.json     --iam-account=540654827891-compute@developer.gserviceaccount.com     

export GOOGLE_APPLICATION_CREDENTIALS=/keys/gsa-key.json    

**ставим HELM**     
https://helm.sh/docs/intro/install/     

curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -  
sudo apt-get install apt-transport-https --yes  
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list  
sudo apt-get update    
sudo apt-get install helm   

Ставим постгрес кластер   

 helm repo add bitnami https://charts.bitnami.com/bitnami     

c помощью helm устанавливаем postgresql      

  helm install pgsql-ha bitnami/postgresql-ha  
```  
 helm install pgsql-ha bitnami/postgresql-ha  
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /keys/kubeconfig.yaml
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /keys/kubeconfig.yaml
NAME: pgsql-ha
LAST DEPLOYED: Thu Mar 25 20:36:09 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

PostgreSQL can be accessed through Pgpool via port 5432 on the following DNS name from within your cluster:

    pgsql-ha-postgresql-ha-pgpool.default.svc.cluster.local

Pgpool acts as a load balancer for PostgreSQL and forward read/write connections to the primary node while read-only connections are forwarded to standby nodes.

To get the password for "postgres" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace default pgsql-ha-postgresql-ha-postgresql -o jsonpath="{.data.postgresql-password}" | base64 --decode)

To get the password for "repmgr" run:

    export REPMGR_PASSWORD=$(kubectl get secret --namespace default pgsql-ha-postgresql-ha-postgresql -o jsonpath="{.data.repmgr-password}" | base64 --decode)

To connect to your database run the following command:

    kubectl run pgsql-ha-postgresql-ha-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql-repmgr:11.11.0-debian-10-r25 --env="PGPASSWORD=$POSTGRES_PASSWORD"  \
        --command -- psql -h pgsql-ha-postgresql-ha-pgpool -p 5432 -U postgres -d postgres

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/pgsql-ha-postgresql-ha-pgpool 5432:5432 &
    psql -h 127.0.0.1 -p 5432 -U postgres -d postgres

```    
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default pgsql-ha-postgresql-ha-postgresql -o jsonpath="{.data.postgresql-password}" | base64 --decode)

root@mgrvkuber:/keys# echo $POSTGRES_PASSWORD

ZH4wv2dw1E

локально фовардим 
kubectl port-forward --namespace default svc/pgsql-ha-postgresql-ha-pgpool 8888:5432

на удаленной машине
kubectl port-forward --address 0.0.0.0 svc/pgsql-ha-postgresql-ha-pgpool  5432:5432

psql -h 10.154.0.6  -U postgres

CREATE ROLE otus LOGIN PASSWORD '1234567890';
CREATE DATABASE otus;
 ALTER DATABASE otus OWNER TO otus;
 \q

добавляем в кластер доступ для otus 
 kubectl exec -it $(kubectl get pods -l app.kubernetes.io/component=pgpool,app.kubernetes.io/name=postgresql-ha -o jsonpath='{.items[0].metadata.name}') -- pg_md5 -m --config-file="/opt/bitnami/pgpool/conf/pgpool.conf" -u "otus" "1234567890"

добавляем чтобы конфиг остался после перезагрузки 

 cat /etc/environment 
export KUBECONFIG=/keys/kubeconfig.yaml   
export GOOGLE_APPLICATION_CREDENTIALS=/keys/gsa-key.json   

