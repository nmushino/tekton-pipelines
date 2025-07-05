
## homeoffice-backend tekton pipeline

![homeoffice-backend-pipeline](../images/homeoffice-backend-pipeline.png)

## Deploy pipelines using kustomize
---
> You may fork this repo and make edit to the `application-deployment/homeoffice/homeoffice-backend/transformer-patches.yaml` in for GitOps or argocd
**Create Projects**
```
oc new-project quarkusdroneshop-cicd
oc new-project quarkusdroneshop-homeoffice
oc adm policy add-role-to-user admin system:serviceaccount:quarkusdroneshop-homeoffice:pipeline -n quarkusdroneshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:quarkusdroneshop-homeoffice -n quarkusdroneshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkusdroneshop-cicd:pipeline -n quarkusdroneshop-homeoffice
```

**Run the kustomize command to deploy pipelines** 
```
kustomize build homeoffice-backend | oc create -f - 
```

**Update Environment Variables in deployment**
```
oc edit deployment.apps/homeoffice-backend
```

## Deploy pipelines Manually 
---
**configure pvc**
```
oc -n quarkusdroneshop-cicd create -f homeoffice-backend/pvc/pvc.yml
oc -n quarkusdroneshop-cicd create -f  ./homeoffice-backend/pvc/maven-source-pvc.yml
```

**configure Tasks**
```
oc -n quarkusdroneshop-cicd create -f ./common-functions/tasks/git-clone.yaml
oc -n quarkusdroneshop-cicd create -f ./common-functions/tasks/openshift-client-task.yaml
oc -n  quarkusdroneshop-cicd create -f ./common-functions/tasks/maven.yaml
```

**Configure push image to quay task**
```
oc -n  quarkusdroneshop-cicd create -f ./homeoffice-backend/tektontasks/pushImageToQuay.yaml
```

**configure Resources**
```
oc -n quarkusdroneshop-cicd create -f  ./homeoffice-backend/resources/git-pipeline-resource.yaml
oc -n quarkusdroneshop-cicd create -f  ./homeoffice-backend/resources/image-pipeline-resource.yaml
```

**Create Pipeline**
```
oc -n quarkusdroneshop-cicd create -f  ./homeoffice-backend/pipeline/deploy-pipeline.yaml
```


### Integration testing instructions 
**Create Projects**
```
oc new-project quarkusdroneshop-cicd
oc new-project quarkusdroneshop-homeoffice
oc adm policy add-role-to-user admin system:serviceaccount:quarkusdroneshop-homeoffice:pipeline -n quarkusdroneshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:quarkusdroneshop-homeoffice -n quarkusdroneshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkusdroneshop-cicd:pipeline -n quarkusdroneshop-homeoffice
```

**Deploy Application**
```
oc create -f application-deployment/homeoffice/homeoffice-backend/ -n quarkusdroneshop-homeoffice
oc expose service/homeoffice-backend -n quarkusdroneshop-homeoffice
```

**Update Environment Variables in deployment**
```
oc edit deployment.apps/homeoffice-backend 
```