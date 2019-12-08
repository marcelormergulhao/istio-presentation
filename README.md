# Istio Demonstration

This repo is guide related to Istio, providing some explanation about different features available.
We use the bookinfo app as part of the setup, and the content is heavily influenced by the tutorials, so if you need/want more reference, please go to the [main project](https://istio.io/).

## Project Setup

First, we need to install docker, minikube and related components. Assuming you have ubuntu 18.04 and no docker/K8S environment already set up:

1. Install docker (via repository):
```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install docker-ce docker-ce-cli
```
  * To test the installation, run `docker run hello-world`
2. Install minikube:
  * If you don't have a hypervisor already, download and install virtualbox
    * Go to [their download page](https://www.virtualbox.org/wiki/Linux_Downloads) and pick the most recent package.
    * Install with `dpkg -i <package_name>` and fix possible issues with `apt install -f`
  * 
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
sudo install minikube /usr/local/bin
```
3. Start the minikube cluster, with memory enhanced
```
minikube start --memory=16384 --cpus=4
```
4. Install Istio and add istioctl to PATH:
```
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.4.1
echo "export PATH=$PWD/bin:$PATH" >> ~/.bashrc
source ~/.bashrc
istioctl manifest apply --set profile=demo
```
5. Enable sidecar injection on the default namespace:
```
kubectl label namespace default istio-injection=enabled
```
6. Deploy the bookinfo app:
```
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```
7. Add gateway to reach bookinfo from outside (with default routes to reach services)
```
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```
8. Set ingress gateway using minikube IP:
```
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
export INGRESS_HOST=$(minikube ip)
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
```
9. Create subsets (destination rules) for bookinfo services
```
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
```

## Demo Scenarios

* Load balancing with round-robin (default)
* Routing based on headers (dev example)
* Canary Deployment
* Fault injection

## Cheat Sheet

* Check istio is running
  * kubectl get pods --namespace istio-system
* Check if sidecar injection is enabled
  * kubectl get namespaces --show-labels
* Check bookinfo is ok
  * kubectl get pods
* Get product page through gateway:
  * curl -s http://${GATEWAY_URL}/productpage
* Kiali username and password
  * admin/admin
* Generate traffic for visualization
  * watch -n 1 curl -o /dev/null -s -w %{http_code} $GATEWAY_URL/productpage
* To open Kiali dashboard
  * istioctl dashboard kiali
* To get Kiali information using the API
  * $KIALI_URL/api/namespaces/graph?namespaces=default&graphType=app