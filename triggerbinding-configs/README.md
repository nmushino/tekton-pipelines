# Create GitHub Trigger for Pipeline

## Create webhook role for permissions 
```
oc -n quarkusdroneshop-cicd create -f ./triggerbinding-configs/webhook-roles.yaml
```
## Generate webhook secret for github
```
WEBHOOK_SECRET="$(openssl rand -base64 12)"
oc -n quarkusdroneshop-cicd create secret generic webhook-secret --from-literal=secret=${WEBHOOK_SECRET}
echo ${WEBHOOK_SECRET} > saved-secert.txt
```