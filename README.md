# redisbank-continuous-deployment

Environment setup:
1. Create a GKE cluster
2. Deploy REC/REDB in three namespaces (development/staging/production)
3. Create Cloud Build triggers for dev/merge/tag
4. Run the demo



#### 1. Create a GKE cluster
```
export PROJECT=$(gcloud info --format='value(config.project)')
export CLUSTER="glau-redisbank-cluster"
export ZONE=us-west1-a

./create_cluster.sh $CLUSTER $ZONE
```


#### 2. Deploy REC/REDB in three namespaces (development/staging/production)
Development GKE Cluster:
```
kubectl create namespace development
kubectl config set-context --current --namespace=development
kubectl apply -f https://raw.githubusercontent.com/RedisLabs/redis-enterprise-k8s-docs/v6.0.20-4/bundle.yaml

kubectl apply -f - <<EOF
apiVersion: app.redislabs.com/v1alpha1
kind: RedisEnterpriseCluster
namespace: development
metadata:
  name: rec
spec:
  nodes: 3
EOF

kubectl apply -f - <<EOF
apiVersion: app.redislabs.com/v1alpha1
kind: RedisEnterpriseDatabase
namespace: development
metadata:
  name: redis-enterprise-database
spec:
  memorySize: 100MB
  modulesList:
    - name: search
      version: 2.0.6
    - name: timeseries
      version: 1.4.8
EOF
```
Staging GKE Cluster:
```
kubectl create namespace staging
kubectl config set-context --current --namespace=staging
kubectl apply -f https://raw.githubusercontent.com/RedisLabs/redis-enterprise-k8s-docs/v6.0.20-4/bundle.yaml

kubectl apply -f - <<EOF
apiVersion: app.redislabs.com/v1alpha1
kind: RedisEnterpriseCluster
namespace: staging
metadata:
  name: rec
spec:
  nodes: 3
EOF

kubectl apply -f - <<EOF
apiVersion: app.redislabs.com/v1alpha1
kind: RedisEnterpriseDatabase
namespace: staging
metadata:
  name: redis-enterprise-database
spec:
  memorySize: 100MB
  modulesList:
    - name: search
      version: 2.0.6
    - name: timeseries
      version: 1.4.8
EOF
```
Production GKE Cluster:
```
kubectl create namespace production
kubectl config set-context --current --namespace=production
kubectl apply -f https://raw.githubusercontent.com/RedisLabs/redis-enterprise-k8s-docs/v6.0.20-4/bundle.yaml

kubectl apply -f - <<EOF
apiVersion: app.redislabs.com/v1alpha1
kind: RedisEnterpriseCluster
namespace: production
metadata:
  name: rec
spec:
  nodes: 3
EOF

kubectl apply -f - <<EOF
apiVersion: app.redislabs.com/v1alpha1
kind: RedisEnterpriseDatabase
namespace: production
metadata:
  name: redis-enterprise-database
spec:
  memorySize: 100MB
  modulesList:
    - name: search
      version: 2.0.6
    - name: timeseries
      version: 1.4.8
EOF
```


#### 3. Create Cloud Build triggers for dev/merge/tag
Create trigger for git dev branch update
```
cat <<EOF > branch-build-trigger.json
{
  "triggerTemplate": {
    "projectId": "${PROJECT}",
    "repoName": "redisbank",
    "branchName": "[^(?!.*master)].*"
  },
  "description": "redisbank-branch",
  "substitutions": {
    "_CLOUDSDK_COMPUTE_ZONE": "${ZONE}",
    "_CLOUDSDK_CONTAINER_CLUSTER": "${CLUSTER}"
  },
  "filename": "cloudbuild-redisbank-dev.yaml"
}
EOF

curl -X POST \
    https://cloudbuild.googleapis.com/v1/projects/${PROJECT}/triggers \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $(gcloud config config-helper --format='value(credential.access_token)')" \
    --data-binary @branch-build-trigger.json
```
Create trigger for git merge with master branch
```
cat <<EOF > master-build-trigger.json
{
  "triggerTemplate": {
    "projectId": "${PROJECT}",
    "repoName": "redisbank",
    "branchName": "master"
  },
  "description": "redisbank-master",
  "substitutions": {
    "_CLOUDSDK_COMPUTE_ZONE": "${ZONE}",
    "_CLOUDSDK_CONTAINER_CLUSTER": "${CLUSTER}"
  },
  "filename": "cloudbuild-redisbank.yaml"
}
EOF

  curl -X POST \
    https://cloudbuild.googleapis.com/v1/projects/${PROJECT}/triggers \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $(gcloud config config-helper --format='value(credential.access_token)')" \
    --data-binary @master-build-trigger.json

```
Create trigger for git tagging a new release
```
cat <<EOF > tag-build-trigger.json
{
  "triggerTemplate": {
    "projectId": "${PROJECT}",
    "repoName": "redisbank",
    "tagName": ".*"
  },
  "description": "redisbank-tag",
  "substitutions": {
    "_CLOUDSDK_COMPUTE_ZONE": "${ZONE}",
    "_CLOUDSDK_CONTAINER_CLUSTER": "${CLUSTER}"
  },
  "filename": "cloudbuild-redisbank-tag.yaml"
}
EOF

curl -X POST \
    https://cloudbuild.googleapis.com/v1/projects/${PROJECT}/triggers \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $(gcloud config config-helper --format='value(credential.access_token)')" \
    --data-binary @tag-build-trigger.json  
```


#### 4. Run the demo
Trigger a cloud-build (redisbank-branch)     
```
git checkout -b dev200
git branch 
vi dummy.txt
git add dummy.txt
git commit -m "dev200"
git push origin dev200
```


Trigger a cloud-build (redisbank-master) by merging changes from dev200  
```
git checkout master
git branch
git merge dev200
git push origin master
```


Trigger a cloud-build (redisbank-master) by merging changes from dev200  
```
git checkout master
git branch
git tag v2.0.1
git push origin v2.0.1
```
