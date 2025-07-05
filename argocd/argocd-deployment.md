# Quarkus Coffeshop and Argocd

## Install the ACM_WORKLOADS option from the deploy-quarkusdroneshop-ansible.sh

```
$ curl -OL https://raw.githubusercontent.com/quarkusdroneshop/quarkusdroneshop-ansible/dev/files/deploy-quarkusdroneshop-ansible.sh
$ chmod +x deploy-quarkusdroneshop-ansible.sh
```
## The script will provide the following
* Gogs server
* OpenShift Pipelines
* OpenShift GitOps
* Quay.io
* AMQ Streams
* Postgres Template deployment
* homeoffice Tekton pipelines
* quarkus-droneshop Tekton pipelines
```
$ cat >env.variables<<EOF
ACM_WORKLOADS=y
AMQ_STREAMS=y
CONFIGURE_POSTGRES=y
MONGODB_OPERATOR=n
MONGODB=n
HELM_DEPLOYMENT=n
EOF
$ ./deploy-quarkusdroneshop-ansible.sh -d ocp4.example.com -t sha-123456789 -p 123456789 -s ATLANTA
```

**Install argocd cli**
```
curl -OL https://github.com/argoproj/argo-cd/releases/download/v2.4.8/argocd-linux-amd64 
sudo mv argocd-linux-amd64 /usr/local/bin/argocd
sudo chmod +x /usr/local/bin/argocd
```

**Login to argocd**
```
nameSpace=openshift-gitops
argoRoute=$(oc get route openshift-gitops-server -n ${nameSpace} -o jsonpath='{.spec.host}')
argoUser=admin
argoPass=$(oc get secret/openshift-gitops-cluster -n ${nameSpace} -o jsonpath='{.data.admin\.password}' | base64 -d)
until [[ $(curl -ks -o /dev/null -w "%{http_code}"  https://${argoRoute}) -eq 200 ]]
do
    sleep 3
    echo -n '.'
done
argocd login --insecure --grpc-web --username ${argoUser} --password ${argoPass} ${argoRoute}

echo "$argoRoute"
echo "$argoPass"
```



## Configure REPO URL
Fork [tekton-pipelines](https://github.com/quarkusdroneshop/tekton-pipelines.git) and update REPO_URL.
```
$ export REPO_URL='https://github.com/quarkusdroneshop/tekton-pipelines.git'
# Example Alternative Repo
REPO_URL='http://gogs-quarkusdroneshop-cicd.apps.cluster-e6dd.e6dd.sandbox568.opentlc.com/user1/tekton-pipelines.git'
```

## HOME Office (Backoffice)
**quarkusdroneshop-homeoffice-ui argo application**  
```
sed "s|%REPO_NAME%|'${REPO_URL}'|g" argocd/quarkusdroneshop-homeoffice-ui/quarkusdroneshop-homeoffice-ui-template.yaml  > argocd/quarkusdroneshop-homeoffice-ui/quarkusdroneshop-homeoffice-ui.yaml
oc create -f argocd/quarkusdroneshop-homeoffice-ui/quarkusdroneshop-homeoffice-ui.yaml  -n openshift-gitops
```

**homeoffice-backend argo application**  
```
sed "s|%REPO_NAME%|'${REPO_URL}'|g" argocd/homeoffice-backend/homeoffice-backend-template.yaml  > argocd/homeoffice-backend/homeoffice-backend.yaml
oc create -f argocd/homeoffice-backend/homeoffice-backend.yaml  -n openshift-gitops
```

**homeoffice-ingress argo application**  
```
sed "s|%REPO_NAME%|'${REPO_URL}'|g" argocd/homeoffice-ingress/homeoffice-ingress-template.yaml  > argocd/homeoffice-ingress/homeoffice-ingress.yaml
oc create -f argocd/homeoffice-ingress/homeoffice-ingress.yaml  -n openshift-gitops
```

## Store front microservices  

**quarkusdroneshop-qdca10 argo application**  
```
sed "s|%REPO_NAME%|'${REPO_URL}'|g" argocd/quarkusdroneshop-qdca10/quarkusdroneshop-qdca10-template.yaml  > argocd/quarkusdroneshop-qdca10/quarkusdroneshop-qdca10.yaml
oc create -f argocd/quarkusdroneshop-qdca10/quarkusdroneshop-qdca10.yaml  -n openshift-gitops
```

**quarkusdroneshop-counter argo application**  
```
sed "s|%REPO_NAME%|'${REPO_URL}'|g" argocd/quarkusdroneshop-counter/quarkusdroneshop-counter-template.yaml  > argocd/quarkusdroneshop-counter/quarkusdroneshop-counter.yaml
oc create -f argocd/quarkusdroneshop-counter/quarkusdroneshop-counter.yaml  -n openshift-gitops
```

**quarkusdroneshop-inventory argo application**  
```
sed "s|%REPO_NAME%|'${REPO_URL}'|g" argocd/quarkusdroneshop-inventory/quarkusdroneshop-inventory-template.yaml  > argocd/quarkusdroneshop-inventory/quarkusdroneshop-inventory.yaml
oc create -f argocd/quarkusdroneshop-inventory/quarkusdroneshop-inventory.yaml  -n openshift-gitops
```


**quarkusdroneshop-qdca10pro argo application**  
```
sed "s|%REPO_NAME%|'${REPO_URL}'|g" argocd/quarkusdroneshop-qdca10pro/quarkusdroneshop-qdca10pro-template.yaml  > argocd/quarkusdroneshop-qdca10pro/quarkusdroneshop-qdca10pro.yaml
oc create -f argocd/quarkusdroneshop-qdca10pro/quarkusdroneshop-qdca10pro.yaml  -n openshift-gitops
```

**quarkusdroneshop-web argo application**   
```
sed "s|%REPO_NAME%|'${REPO_URL}'|g" argocd/quarkusdroneshop-web/quarkusdroneshop-web-template.yaml  > argocd/quarkusdroneshop-web/quarkusdroneshop-web.yaml
oc create -f argocd/quarkusdroneshop-web/quarkusdroneshop-web.yaml  -n openshift-gitops
```

## RHEL Edge Pipelines
**quarkusdroneshop-majestic-monolith argo application**   
```
sed "s|%REPO_NAME%|'${REPO_URL}'|g" argocd/quarkusdroneshop-majestic-monolith/edge-application-template.yaml > argocd/quarkusdroneshop-majestic-monolith/edge-application.yaml

oc create -f argocd/quarkusdroneshop-majestic-monolith/edge-application.yaml  -n quarkusdroneshop-cicd
```


