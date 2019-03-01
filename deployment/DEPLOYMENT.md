### K8S deployment
**[Docker](#docker)**<br>
**[GKE](#gke)**<br>
**[Terraform](#terraform)**<br>
**[Pulumi](#pulumi)**<br>

Docker
------

**(!)** As a prerequisite you need to build docker image and publish it to your favorite
container repository. Example for [GCR](https://cloud.google.com/container-registry/) repository: 

- build the image:
```js
$ git clone git@github.com:jkogut/simple-python-rest-api-v1.git
$ sudo docker build --tag simple_python_rest_api:1.0.0 .
```
- configure docker to access GCR, retag and push the image:
```js
$ gcloud auth configure-docker
$ export PROJECT_ID="$(gcloud config get-value project -q)"
$ sudo docker tag simple_python_rest_api:1.0.0 gcr.io/${PROJECT_ID}/simple_python_rest_api:1.0.0
$ sudo docker push gcr.io/${PROJECT_ID}/simple_python_rest_api:1.0.0
```

GKE
---

Create K8S cluster using **GKE**:
```js
$ gcloud container clusters create py-rest-api-v1 --num-nodes=3
$ kubectl run simple-python-rest-api --image=gcr.io/${PROJECT_ID}/simple_python_rest_api:1.0.0 --port 5002
$ export DEP=$(kubectl get deployments | tail -1 | awk '{print $1}')
$ kubectl expose deployment $DEP --type=LoadBalancer --port 80 --target-port 5002
```

- wait a little for **EXTERNAL-IP** assignment:
```js
$ kubectl get service
NAME                     TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)        AGE
kubernetes               ClusterIP      10.11.240.1   <none>           443/TCP        18m
simple-python-rest-api   LoadBalancer   10.11.245.2   35.204.242.122   80:31100/TCP   1m
```

- test with curl
```js
$ export DEP_IP=$(kubectl get service $DEP | tail -1 | awk '{print $4}')
$ curl -s http://${DEP_IP}/api/status |jq
$ curl -s http://${DEP_IP}/api/v1/employees/1 |jq
```

Delete **LoadBalancer** service and cluster:

```js
$ kubectl delete service $DEP
$ gcloud container clusters delete py-rest-api-v1
```