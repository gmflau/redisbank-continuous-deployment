# redisbank-continuous-deployment

Environment setup:
1. Create a GKE cluster
2. Create Cloud Build triggers for dev/merge/tag
3. Run the demo



#### 1. Create a GKE cluster
```
./create-cluster glau-redisbank-cluster us-west1-a

export PROJECT=$(gcloud info --format='value(config.project)')
export ZONE=us-west1-a
export CLUSTER=glau-redisbank-cluster
```

#### 2. Create Cloud Build triggers for dev/merge/tag
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


#### 3. Run the demo
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
