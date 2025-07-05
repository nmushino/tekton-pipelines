# quarkusdroneshop-inventory Tekton pipeline
![quarkusdroneshop-inventory](../images/quarkusdroneshop-inventory.png)

## Deploy pipelines using Kustomize
> You may fork this repo and make edit to the `application-deployment/store/quarkusdroneshop-inventory/transformer-patches.yaml` in for GitOps or argocd
---
**Create Projects and configure permissions**
```
oc new-project quarkusdroneshop-cicd
oc new-project quarkusdroneshop-demo
oc adm policy add-role-to-user admin system:serviceaccount:quarkusdroneshop-demo:pipeline -n quarkusdroneshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:quarkusdroneshop-demo -n quarkusdroneshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkusdroneshop-cicd:pipeline -n quarkusdroneshop-demo
```
**Run the Kustomize command to deploy pipelines** 
```
kustomize build quarkusdroneshop-inventory | oc create -f - 
```

**Update Environment Variables in deployment**
```
oc edit deployment.apps/quarkusdroneshop-inventory  -n quarkusdroneshop-demo
```

## Deploy pipelines Manually 
---
**configure pvc**
```
oc -n quarkusdroneshop-cicd create -f quarkusdroneshop-inventory/pvc/pvc.yml
oc -n quarkusdroneshop-cicd create -f  ./quarkusdroneshop-inventory/pvc/maven-source-pvc.yml
```


**configure Tasks**
```
oc -n quarkusdroneshop-cicd create -f ./common-functions/tasks/git-clone.yaml
oc -n quarkusdroneshop-cicd create -f ./common-functions/tasks/openshift-client-task.yaml
oc -n  quarkusdroneshop-cicd create -f ./common-functions/tasks/maven.yaml
```

**Configure push image to quay task**
```
oc -n  quarkusdroneshop-cicd create -f ./quarkusdroneshop-inventory/tektontasks/pushImageToQuay.yaml
```

**configure Resources**
```
oc -n quarkusdroneshop-cicd create -f  ./quarkusdroneshop-inventory/resources/git-pipeline-resource.yaml
oc -n quarkusdroneshop-cicd create -f  ./quarkusdroneshop-inventory/resources/image-pipeline-resource.yaml
```

**Create Pipeline**
```
oc -n quarkusdroneshop-cicd create -f  ./quarkusdroneshop-inventory/pipeline/deploy-pipeline.yaml
```


### Integration testing instructions 
```
oc adm policy add-role-to-user admin system:serviceaccount:quarkusdroneshop-demo:pipeline -n quarkusdroneshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:quarkusdroneshop-demo -n quarkusdroneshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkusdroneshop-cicd:pipeline -n quarkusdroneshop-demo

oc project quarkusdroneshop-demo
oc create -f application-deployment/store/quarkusdroneshop-qdca10pro/quarkusdroneshop-qdca10pro.yaml -n quarkusdroneshop-demo
```