
## SETUP NGINX Ingress Controller


We'll start with the documentation as always
You can find the [Kubernetes NGINX documentation here](https://kubernetes.github.io/ingress-nginx/) documentation here
**Official Page** - https://kubernetes.github.io/ingress-nginx/deploy/

First thing we do is check the compatibility matrix to ensure we are deploying a compatible version of NGINX Ingress on our Kubernetes cluster


The Documentation also has a link to the GitHub Repo which has a compatibility matrix 

kubectl get nodes
### Install Git if not already installed
yum install git -y

### Clone thhe IngressController Repo from official git page
git clone https://github.com/nginxinc/kubernetes-ingress/

### Get the installation YAML

The controller ships as a helm chart, so we can grab version v1.7.0 as per the compatibility matrix.
From our container we can do this:

```shell
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm search repo ingress-nginx --versions
```


From the app version we select the version that matches the compatibility matrix.
```shell
NAME                            CHART VERSION   APP VERSION     DESCRIPTION
ingress-nginx/ingress-nginx     4.6.0           1.7.0           Ingress controller for Kubernetes using NGINX a...
```

Now we can use helm to install the chart directly if we want.
Or we can use helm to grab the manifest and explore its content.
We can also add that manifest to our git repo if we are using a GitOps workflow to deploy it.

```shell
CHART_VERSION="4.6.0"
APP_VERSION="1.7.0"

mkdir ./root/nginx-deploy-1.7.0

helm template ingress-nginx ingress-nginx \
--repo https://kubernetes.github.io/ingress-nginx \
--version ${CHART_VERSION} \
--namespace ingress-nginx \
> ./root/nginx-deploy-1.7.0
```

### Deploy the Ingress controller

```shell
kubectl create namespace ingress-nginx
kubectl apply -f ./kubernetes/ingress/controller/nginx/manifests/nginx-ingress.${APP_VERSION}.yaml
```


### Check the installation

```shell
kubectl -n ingress-nginx get pods
```


The traffic for our cluster will come in over the Ingress service
Note that we dont have load balancer capability in kind by default, so our LoadBalancer is pending:

```shell
kubectl -n ingress-nginx get svc
NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.96.130.21    <pending>     80:31011/TCP,443:31772/TCP   26m
ingress-nginx-controller-admission   ClusterIP      10.96.125.210   <none>        443/TCP                      26m
```

For testing purposes, we will simply setup port-forwarding
If you are running in the cloud, you will get a real IP address.

```shell
kubectl -n ingress-nginx port-forward svc/ingress-nginx-controller 443
```

We can reach our controller on https://localhost/

It's important to understand that Ingress runs on two ports 80 and 443
NGINX Ingress creates a fake certificate which is served for default HTTPS traffic on port 443.
If you look in the browser you will notice the name of the certificate Common Name (CN)	Kubernetes Ingress Controller Fake Certificate


