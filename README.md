## Create cluster with Service Accounts


```shell
export ZONE=us-west1-b
export NODE_SA_NAME=gke-node-sa
gcloud iam service-accounts create  $NODE_SA_NAME --display-name "Node Service Account"
export NODE_SA_EMAIL=`gcloud iam service-accounts list --format='value(email)' --filter='displayName:Node Service Account'`
export PROJECT=`gcloud config get-value project`
gcloud projects add-iam-policy-binding $PROJECT --member=serviceAccount:${NODE_SA_EMAIL} --role=roles/monitoring.metricWriter
gcloud projects add-iam-policy-binding $PROJECT --member=serviceAccount:${NODE_SA_EMAIL} --role=roles/monitoring.viewer
gcloud projects add-iam-policy-binding $PROJECT --member=serviceAccount:${NODE_SA_EMAIL} --role=roles/logging.logWriter
gcloud config set container/new_scopes_behavior true
gcloud config set container/use_v1_api false
```

## workload-metadata-from-node=SECURE

```shell
gcloud beta container clusters create conceal-1-9 --service-account=$NODE_SA_EMAIL  --workload-metadata-from-node=SECURE --zone=$ZONE  --cluster-version=1.9
```

## Test default/identity

access `default/identity` from the node

```console
user@cloud-shell $ gcloud compute ssh gke-conceal-1-9-default-pool-8bb964a5-8rhd
curl -H "Metadata-Flavor: Google" 'http://metadata/computeMetadata/v1/instance/service-accounts/default/identity?audience=https://www.example.com'
<TOKEN>
```
## Test metadata

test regular medata access from pod

```console
user@cloud-shell $ gcloud container clusters get-credentials conceal-1-9 --zone $ZONE && kubectl run -it --rm my-shell --image nginx -- bash
apt-get update -qy && apt-get install -qy curl && curl metadata.google.internal/computeMetadata/v1/instance/name -i -H 'Metadata-Flavor:Google' && echo $?

HTTP/1.1 200 OK
Content-Length: 42
Content-Type: application/text
Date: Sun, 27 May 2018 14:28:57 GMT
Etag: 630ddb9980ea9e3
Metadata-Flavor: Google
Server: Metadata Server for VM
X-Frame-Options: SAMEORIGIN
X-Xss-Protection: 1; mode=block
gke-conceal-1-9-default-pool-8bb964a5-8rhd0
```

## Test concealment

test concealed metadata from the pod

```console
root@pod# curl -H "Metadata-Flavor: Google" 'http://metadata/computeMetadata/v1/instance/service-accounts/default/identity?audience=https://www.example.com'
This metadata endpoint is concealed.
```

