# Airflow Example Values.yaml

Please note this Values.yaml works only in tandem with [ this AIRFLOW Helm chart ](https://github.com/helm/charts/tree/master/stable/airflow)

Working for -
   - App Version: 1.10.10
   - Chart Version: 7.1.5
---
## Salient Features
- Kubernetes Executor
- S3/S3 compatible storage for logging
- git auto-syncup 
- Auto installation of  pip dependencies on Scheduler and Web pods
---
## Configuration
Please modify the following Values to get your chart running in no time!
```
# this is a set of ENV variables that need to be modified in Airflow - config section
# this is Airflow related configuration, more info can be found at -
# official webiste -  https://airflow.apache.org/docs/stable/configurations-ref.html

airflow:
  config: 
    AIRFLOW__KUBERNETES__NAMESPACE: MyK8sNamespaceName
    AIRFLOW__KUBERNETES__WORKER_SERVICE_ACCOUNT_NAME : MyK8sNamespaceServiceAccountName 
    AIRFLOW__KUBERNETES__GIT_REPO: https://username@password:mygitserver.domain
    #please refer below for custom image in the "Note section"
    AIRFLOW__KUBERNETES__WORKER_CONTAINER_REPOSITORY: repo_path_for_custom_Docker_Image_for_Workers
    AIRFLOW__KUBERNETES__GIT_BRANCH: master
    AIRFLOW__CORE__REMOTE_BASE_LOG_FOLDER: s3://mybucketname/myfolder


# dag path must be same as reponame 
dags:
  - path: myreponame
# define your git url and branch 
# please note you can also use private repos with Deploy tokens

git: 
  url: https://username@password:mygitserver.domain
  branch: master

# depending on your k8s implementation add respective Annotations
# also you can give any name for your ingress host
ingress: 
  annotations:
  host:  what-you-wannaname.your.domain
# this is the SA your KubernetesExecutor will use for spawning new pods for each task
# please note, it is expected to have  your SA secrets in default location "/var/...."
serviceAccount: 
  name: my-k8s-serviceAccountName
```

---
## NOTE 
One catch in this implementation that is kind of wierd is that the worker pods, do not automatically 
have the pip dependencies installed, to over come this challenge, you might want to have  your own version 
Dockerimage referenced for Worker pods

A sample Dockerfile has been provided:

In  your docker host, create a file with name Dockerfile
```shell script
vi Dockerfile
```
and then, copy the below lines and append your dependencies to run command
```Dockerfile
from apache/airflow:1.10.10-python3.6
#you can add your other dependencies here 
run  pip install --user  pymongo requests redis 
```
After saving the above dockerfile, you build and push to your repo

```shell script
docker build -t my_worker_image .
docker tag my_worker_image my_username:my_worker_image
#after this please validate the image and then, login to your repo,
docker login 
#enter your credentials and, push your image
docker push  my_username:my_worker_image
```
