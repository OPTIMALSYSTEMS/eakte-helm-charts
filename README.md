# eakte helm charts
## eakte installation

First please add your credentials for the docker.yuuvis.org registry in the values yaml files of the helm charts.

Replace all **changeme** default passwords in the values.yaml of the charts you plan to use.  

### Add required Helm repositorys:

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
```
### Install the eakte Helm chart

#### Update infrastructure dependencies

```shell
cd eakte
helm dep up
cd ..
```

#### Edit the infrastructure values.yaml

* Edit the docker registry credentials. 
* Optionally change passwords
* Optionally change the used storage classes

#### Install eakte chart

```shell
kubectl create namespace eakte
helm install eakte ./eakte --namespace eakte
```
Check if the pods are deployed and ready. It should look like this:

kubectl get pods -n eakte

```shell
NAME                                       READY   STATUS    RESTARTS   AGE
eakte-api-8d9cf8797-thj5m                  1/1     Running   0          1m
eakte-client-5d69f8c5d6-6ddhd              1/1     Running   0          1m
eakte-fileplan-importer-547c9d5487-bhgbx   1/1     Running   0          1m
eakte-reverse-proxy-687d6c655f-h7nlr       1/1     Running   0          1m
eakte-webhook-6764987879-9wdzm             1/1     Running   0          1m
redis-master-0                             1/1     Running   0          1m
```

## Install eakte App in yuuvis Momentum
1. Port Forward the yuuvis configuration storage gogs to the local port 3000:
```kubectl port-forward service/gogs -n infrastructure 3000:80```
2. Open the URL localhost:3000 in a browser.
3. Enter your gogs Credentials which you have defined in the helm values of the infrastructure chart.

Example from infracstructure values.yaml:
```yaml
gogs:
  git:
    user: yuuvis
    password: changeme
```

1. Login and open the yuuvis-config repository.
2. Open authentication-prod.yml
3. Click the pen icon to edit the file.
4. Replace the routing.defaultEntryPoint with ```routing.defaultEntryPoint: '/eakte/index.html'```
5. Add eakte to the routing.endpoints
6. Add to authorization.accesses
```yaml
### eakte-client
  - endpoints: /eakte/**
```
7. Save changes.
8. Open system/systemHookConfiguration.json
9. Add the following to the webhooks array:
Replace .eakte with the correct namespace if the eakte is not installed in the eakte namespace:
```json
 {
			"enable" : true,
			"predicate" : "spel:(options==null || options['currentVersion']==null) && (properties['system:objectTypeId']['value']=='appEakte:record')",
			"type" : "dms.request.import.storage.before",
			"url" : "http://eakte-webhook.eakte/createRecord",
			"useDiscovery" : true
		},
        {
			"enable" : true,
			"predicate" : "spel:(options==null || options['currentVersion']==null) && (properties['system:objectTypeId']['value']=='appEakte:recordFolder')",
			"type" : "dms.request.import.storage.before",
			"url" : "http://eakte-webhook.eakte/createRecordFolder",
			"useDiscovery" : true
		},
        {
			"enable" : true,
			"predicate" : "spel:(options==null || options['currentVersion']==null) && (properties['system:objectTypeId']['value']=='appEakte:recordCategory')",
			"type" : "dms.request.import.storage.before",
			"url" : "http://eakte-webhook.eakte/createRecordCategory",
			"useDiscovery" : true
		},
        {
			"enable" : true,
			"predicate" : "spel:(options==null || options['currentVersion']==null) && (properties['system:objectTypeId']['value']=='appEakte:eAkteSettings')",
			"type" : "dms.request.import.storage.before",
			"url" : "http://eakte-webhook.eakte/createEAkteSettings",
			"useDiscovery" : true
		},
        {
			"enable" : true,
			"predicate" : "spel:(options==null || options['currentVersion']==null) && (properties['system:objectTypeId']['value']=='appEakte:logo')",
			"type" : "dms.request.import.storage.before",
			"url" : "http://eakte-webhook.eakte/createLogo",
			"useDiscovery" : true
		},
        {
			"enable" : true,
			"predicate" : "spel:(properties['system:objectTypeId']['value']=='appEakte:filePlan')",
			"type" : "dms.request.import.storage.before",
			"url" : "http://eakte-webhook.eakte/refresh/fileplan",
			"useDiscovery" : true
		}
```
10. Restart yuuvis repository
11. kubectl -n yuuvis rollout restart deployment repository

## Upload schema
Replace inserttenant with your tenant. name with your username and password with your password.
```bash
curl --location --request POST 'https://yoururl/api/system/apps/eakte/schema' \
--header 'X-ID-TENANT-NAME: inserttenant' \
--header 'Content-Type: application/xml' \
--user name:password \
--data-binary '@/pathToEAkteSchema'
```

## Import FilePlan
### Convert FilePlan
Replace inserttenant with your tenant. name with your username and password with your password.
```bash
curl --location --request POST 'https://yoururl/eakte/fileplan-importer-api/aktenplan21ToFileplan' \
--header 'X-ID-TENANT-NAME: inserttenant' \
--user name:password \
--form 'file=@/fileplanpath' >> fileplan.json
```

### Upload FilePlan
Replace path to fileplan with the path of the converted fileplan, inserttenant with 
your tenant and username and password with your username and password and yoururl with the url of your cluster,
fileplantitle with your filePlan title:

```bash
curl --location --request POST 'https://yoururl/api/dms/objects' \
--header 'X-ID-TENANT-NAME: inserttenant' \
--header 'Content-Type: multipart/form-data' \
--user name:password \
--form 'data="{
  \"objects\": [{
    \"properties\": {
      \"system:objectTypeId\": {
        \"value\": \"fileplan\"
      },
      \"filePlanTitle\": {
        \"value\": \"fileplantitle\"
      } 
    },
    \"contentStreams\": [{
      \"cid\": \"fileplan\",
      \"mimeType\": \"application/json\",
      \"fileName\" : \"fileplan.json\"
    }]
  }]
}";type=application/json' \
--form 'fileplan=@"pathtofileplan"'
```

### Viewer
add to authorization.accesses in authentication-prod.yml
```
### Webclient
  - endpoints: /viewer/**
```