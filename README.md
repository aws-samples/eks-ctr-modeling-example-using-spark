## Docker
### Login to Amazon ECR registry (Optional)
See https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecr/get-login-password.html

### Build Docker
Following image can be used as ```spark.kubernetes.container.image``` in the tutorial notebook.

```
cd docker/spark
export SPARK_IMAGE=<AWS_ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com/<ECR-REPO>:spark
docker build -t $SPARK_IMAGE .
docker push $SPARK_IMAGE
```

Following image can be used in ```jupyter-notebook.yaml```.
```
cd docker/jupyter
export JUPYTER_IMAGE=<AWS_ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com/<ECR-REPO>:jupyter
docker build -t $JUPYTER_IMAGE .
docker push $JUPYTER_IMAGE
```

## Kubernetes
### Create Namespace

Before following the tutorial, let's set up and start a Jupyter notebook server in your EKS cluster. First, let's create a namespace called ```notebook```.
```
kubectl apply -f yaml/namespace.yaml
```

### Create Spark ServiceAccount

Then we can create a ```ClusterRoleBinding``` and ```ServiceAccount```, which we will use to run Spark on Kubernetes.
```
kubectl apply -f yaml/spark-rbac.yaml
```

### Create Jupyter notebook server

Finally, we will use the docker image provided in this tutorial to deploy notebook server as a ```Deployment``` and build ```Service``` on top of it to expose IPs for notebook UI and Spark UI.
```
kubectl apply -f yaml/jupyter-notebook.yaml
```

### Access jupyter notebook via browser

You can check the status of your service via:
```
kubectl -n notebook get service
```

Once your service is up, you are ready to access your Jupyter notebook via and open http://localhost:8888
```
kubectl -n notebook port-forward service/jupyter-svc 8888:80
```