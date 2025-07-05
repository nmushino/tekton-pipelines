## quarkusdroneshop-qdca10 tekton pipeline

![quarkusdroneshop-qdca10](../images/quarkusdroneshop-qdca10.png)

## Deploy pipelines using kustomize
> You may fork this repo and make edit to the `application-deployment/store/quarkusdroneshop-qdca10/transformer-patches.yaml` in for GitOps or argocd
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
kustomize build quarkusdroneshop-qdca10 | oc create -f - 
```

**Update Environment Variables in deployment**
```
oc edit deployment.apps/quarkusdroneshop-qdca10  -n quarkusdroneshop-demo
```


## Deploy pipelines Manually 
---
**configure pvc**
```
oc -n quarkusdroneshop-cicd create -f quarkusdroneshop-qdca10/pvc/pvc.yml
oc -n quarkusdroneshop-cicd create -f  ./quarkusdroneshop-qdca10/pvc/maven-source-pvc.yml
```


**configure Tasks**
```
oc -n quarkusdroneshop-cicd create -f ./common-functions/tasks/git-clone.yaml
oc -n quarkusdroneshop-cicd create -f ./common-functions/tasks/openshift-client-task.yaml
oc -n  quarkusdroneshop-cicd create -f ./common-functions/tasks/maven.yaml
```

**Configure push image to quay task**
```
oc -n  quarkusdroneshop-cicd create -f ./quarkusdroneshop-qdca10/tektontasks/pushImageToQuay.yaml
```

**configure Resources**
```
oc -n quarkusdroneshop-cicd create -f  ./quarkusdroneshop-qdca10/resources/git-pipeline-resource.yaml
oc -n quarkusdroneshop-cicd create -f  ./quarkusdroneshop-qdca10/resources/image-pipeline-resource.yaml
```

**Create Pipeline**
```
oc -n quarkusdroneshop-cicd create -f  ./quarkusdroneshop-qdca10/pipeline/deploy-pipeline.yaml
```


### Integration testing instructions 
```
oc adm policy add-role-to-user admin system:serviceaccount:quarkusdroneshop-demo:pipeline -n quarkusdroneshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:quarkusdroneshop-demo -n quarkusdroneshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkusdroneshop-cicd:pipeline -n quarkusdroneshop-demo

oc project quarkusdroneshop-demo
oc create -f application-deployment/store/quarkusdroneshop-qdca10/quarkusdroneshop-qdca10.yaml -n quarkusdroneshop-demo
```