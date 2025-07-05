## quarkusdroneshop-customermocker tekton pipeline


## Deploy pipelines using kustomize
> You may fork this repo and make edit to the `application-deployment/store/quarkusdroneshop-customermocker/transformer-patches.yaml` in for GitOps or argocd
---
**Set target project**
```
export TARGET_PROJECT=quarkusdroneshop-demo
or 
export TARGET_PROJECT=quarkusdroneshop-integration
```

**Create Projects and configure permissions**
```
oc new-project quarkusdroneshop-cicd
oc new-project ${TARGET_PROJECT}
oc adm policy add-role-to-user admin system:serviceaccount:${TARGET_PROJECT}:pipeline -n quarkusdroneshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:${TARGET_PROJECT} -n quarkusdroneshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkusdroneshop-cicd:pipeline -n ${TARGET_PROJECT}
```

**Update the kustomization.yaml with TARAGE_PROJECT name**
```
 sed -i "s/^namespace:.*/namespace: ${TARGET_PROJECT}/" application-deployment/store/quarkusdroneshop-customermocker/kustomization.yaml
```

**Run the kustomize command to deploy pipelines** 
```
kustomize build quarkusdroneshop-customermocker | oc create -f - 
```

**Update Environment Variables in deployment**
```
oc edit deployment.apps/quarkusdroneshop-customermocker  -n ${TARGET_PROJECT}
```


## Deploy pipelines Manually 
---
**configure pvc**
```
oc -n quarkusdroneshop-cicd create -f quarkusdroneshop-customermocker/pvc/pvc.yml
oc -n quarkusdroneshop-cicd create -f  ./quarkusdroneshop-customermocker/pvc/maven-source-pvc.yml
```


**configure Tasks**
```
oc -n quarkusdroneshop-cicd create -f ./common-functions/tasks/git-clone.yaml
oc -n quarkusdroneshop-cicd create -f ./common-functions/tasks/openshift-client-task.yaml
oc -n  quarkusdroneshop-cicd create -f ./common-functions/tasks/maven.yaml
```

**Configure push image to quay task**
```
oc -n  quarkusdroneshop-cicd create -f ./quarkusdroneshop-customermocker/tektontasks/pushImageToQuay.yaml
```

**configure Resources**
```
oc -n quarkusdroneshop-cicd create -f  ./quarkusdroneshop-customermocker/resources/git-pipeline-resource.yaml
oc -n quarkusdroneshop-cicd create -f  ./quarkusdroneshop-customermocker/resources/image-pipeline-resource.yaml
```

**Create Pipeline**
```
oc -n quarkusdroneshop-cicd create -f  ./quarkusdroneshop-customermocker/pipeline/deploy-pipeline.yaml
```


### Integration testing instructions 

**Set target project**
```
export TARGET_PROJECT=quarkusdroneshop-demo
or 
export TARGET_PROJECT=quarkusdroneshop-integration
```

**Create Projects and configure permissions**
```
oc new-project quarkusdroneshop-cicd
oc new-project ${TARGET_PROJECT}
oc adm policy add-role-to-user admin system:serviceaccount:${TARGET_PROJECT}:pipeline -n quarkusdroneshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:${TARGET_PROJECT} -n quarkusdroneshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkusdroneshop-cicd:pipeline -n ${TARGET_PROJECT}
```

**Deploy application**
```
oc project quarkusdroneshop-demo
oc create -f application-deployment/store/quarkusdroneshop-customermocker/ -n quarkusdroneshop-demo
oc expose service/quarkusdroneshop-customermocker -n quarkusdroneshop-demo
```