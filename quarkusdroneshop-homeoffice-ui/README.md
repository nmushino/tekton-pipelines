
## quarkusdroneshop-homeoffice-ui tekton pipeline

![homeoffice-dashboard](../images/homeoffice-dashboard.png)
![homeoffice-ui-pipeline](../images/homeoffice-ui-pipeline.png)

**Create Projects**
```
oc new-project quarkusdroneshop-cicd
oc new-project quarkusdroneshop-homeoffice
oc adm policy add-role-to-user admin system:serviceaccount:quarkusdroneshop-homeoffice:pipeline -n quarkusdroneshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:quarkusdroneshop-homeoffice -n quarkusdroneshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkusdroneshop-cicd:pipeline -n quarkusdroneshop-homeoffice
```

## Deploy pipelines using kustomize
---

**Run the kustomize command to deploy pipelines** 
```
kustomize build quarkusdroneshop-homeoffice-ui | oc create -f - 
```

## Deploy pipelines Manually 
---
**configure Tasks**
```
oc -n quarkusdroneshop-cicd create -f ./quarkusdroneshop-homeoffice-ui/tektontasks/s2i-nodejs-task.yaml
oc -n  quarkusdroneshop-cicd create -f ./quarkusdroneshop-homeoffice-ui/tektontasks/pushImageToQuay.yaml
```

**configure Resources**
```
oc -n quarkusdroneshop-cicd create -f  ./quarkusdroneshop-homeoffice-ui/resources/image-pipeline-resource.yaml
oc -n quarkusdroneshop-cicd create -f  ./quarkusdroneshop-homeoffice-ui/resources/git-pipeline-resource.yaml
```

**Create Pipeline**
```
oc -n quarkusdroneshop-cicd create -f  ./quarkusdroneshop-homeoffice-ui/pipeline/deploy-pipeline.yaml
```

## For Git WebHooks
---
**Create GitHub Trigger for Pipeline**
```
oc -n quarkusdroneshop-cicd create -f ./triggerbinding-configs/webhook-roles.yaml
oc -n quarkusdroneshop-cicd create -f ./triggerbinding-configs/github-triggerbinding.yaml
WEBHOOK_SECRET="$(openssl rand -base64 12)"
oc -n quarkusdroneshop-cicd create secret generic webhook-secret --from-literal=secret=${WEBHOOK_SECRET}
echo ${WEBHOOK_SECRET} > saved-secert.txt
sed -i "s/<git-triggerbinding>/github-triggerbinding/" ./quarkusdroneshop-homeoffice-ui/webhook.yaml
sed -i "/ref: github-triggerbinding/d" ./quarkusdroneshop-homeoffice-ui/webhook.yaml
sed -i "s/- name: pipeline-binding/- name: github-triggerbinding/" ./quarkusdroneshop-homeoffice-ui/webhook.yaml
oc -n quarkusdroneshop-cicd create -f  ./quarkusdroneshop-homeoffice-ui/webhook.yaml
```

**Create HomeOffice Webhook**
```
oc -n quarkusdroneshop-cicd create route edge homeoffice-ui-webhook --service=el-homeoffice-ui-webhook --port=8080 --insecure-policy=Redirect

oc -n quarkusdroneshop-cicd  get route homeoffice-ui-webhook -o jsonpath='https://{.spec.host}'

```

**Configure web hook for quarkusdroneshop-homeoffice-ui**

    > **NOTE**: Every Git server has its own properties, but basically you want to provide the ingress url for our webhook and when the Git server should send the hook. E.g: push events, PR events, etc.

    1. Go to your application repository on GitHub, eg: https://github.com/quarkusdroneshop/quarkusdroneshop-homeoffice-ui
    2. Click on `Settings` -> `Webhooks`
    3. Create the following `Hook`
       1. `Payload URL`: Output of command `oc -n quarkusdroneshop-cicd  get route homeoffice-ui-webhook -o jsonpath='https://{.spec.host}'`
       2. `Content type`: application/json
       2. `Secret`: v3r1s3cur3 `cat saved-secert.txt`
       3. `Events`: Check **Push Events**, leave others blank
       4. `Active`: Check it
       5. `SSL verification`: Check  **Disable**
       6. Click on `Add webhook`


### Integration testing instructions 
```
oc new-project quarkusdroneshop-homeoffice
oc adm policy add-role-to-user admin system:serviceaccount:quarkusdroneshop-homeoffice:pipeline -n quarkusdroneshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:quarkusdroneshop-homeoffice -n quarkusdroneshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkusdroneshop-cicd:pipeline -n quarkusdroneshop-homeoffice

oc project quarkusdroneshop-homeoffice
oc new-app quarkusdroneshop-cicd/quarkusdroneshop-homeoffice-ui:latest -n quarkusdroneshop-homeoffice
oc expose svc quarkusdroneshop-homeoffice-ui -n quarkusdroneshop-homeoffice
```

