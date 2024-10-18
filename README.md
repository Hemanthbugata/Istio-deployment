# Istio Project

## Step 1 - EC2 Instance Setup

1. Launch an EC2 instance with **Amazon Linux 2 AMI**.
2. Allocate **20 GB** of storage.

## Step 2 - Install Minikube and Docker

Run the following commands on your EC2 instance:

```
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo yum install conntrack -y
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
/usr/local/bin/minikube start --force --driver=docker
```

## Step 3 - Install Kubectl 

```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.20.4/2021-04-12/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin
cp ./kubectl $HOME/bin/kubectl
export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source $HOME/.bashrc
kubectl version --short --client
```

## Step 4 - Install Istio

```
cd /opt/
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.23.2
export PATH=$PWD/bin:$PATH
istioctl install -f samples/bookinfo/demo-profile-no-gateways.yaml -y
kubectl label namespace default istio-injection=enabled
sudo yum install git -y

kubectl get crd gateways.gateway.networking.k8s.io &> /dev/null || { \
kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v1.1.0" | kubectl apply -f -; }
```

## Step 5 - make a deployment 

```
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.23/samples/bookinfo/platform/kube/bookinfo.yaml
kubectl get services
kubectl get pods

kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- \
curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
```

## Step 6 - Create a Virtual Gateway

```
kubectl apply -f samples/bookinfo/gateway-api/bookinfo-gateway.yaml
kubectl annotate gateway bookinfo-gateway networking.istio.io/service-type=ClusterIP --namespace=default
kubectl get gateway
kubectl port-forward svc/bookinfo-gateway-istio 8080:80
```
## Step 7 - Port forward kiali 

```
kubectl apply -f samples/addons
kubectl rollout status deployment/kiali -n istio-system
kubectl port-forward --address 0.0.0.0 svc/kiali 9008:20001 -n istio-system
```
 login with <Instance ip: 9008 > to access kiali dashboard 
 
## Step 8 - Port forward Product page 

```
kubectl port-forward --address 0.0.0.0 svc/productpage 9010:9080
kubectl port-forward svc/bookinfo-gateway-istio 8080:80

```
login with <instance ip : 9010 > to access product page 

## OUTPUT FIlES 

  <h2>KIALI DASHBOARD </h2>

      ![P1](https://github.com/user-attachments/assets/424e3dbe-8910-4a06-9fdd-3935cfb12272)

  <h3>Metrics and Mesh </h3>
  
      ![P3](https://github.com/user-attachments/assets/f9b3a6ec-0f5c-462f-a1f3-de7d3cab0749)


  <h2>Product Page </h2>

       ![P2](https://github.com/user-attachments/assets/6d89ae3f-ef94-4b0a-a203-3849b81d0025)

