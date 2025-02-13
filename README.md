

# microk8s - Bare Metal Quick Install
Build Process Cheat Sheet  

 

![download](https://user-images.githubusercontent.com/993459/111545821-ea5d5880-8733-11eb-9352-d22f812e9fb0.png)

## Base Install - Ubuntu 20.x
```
sudo snap install microk8s --classic --channel=1.25/stable
```
```
sudo microk8s status --wait-ready
```
```
sudo microk8s enable dashboard dns registry helm3 ingress prometheus
```
```
sudo snap alias microk8s.kubectl kubectl    
```
1. After the above install, you will get the message. re-login to have the alias enabled
```
kubectl
Insufficient permissions to access MicroK8s.
You can either try again with sudo or add the user cloud_user to the 'microk8s' group:

    sudo usermod -a -G microk8s cloud_user
    sudo chown -f -R cloud_user ~/.kube

The new group will be available on the user's next login.
```

## Install Helm
[Helm Install](https://helm.sh/docs/intro/install/)

## Ingress Install - Ubuntu 20.x

1. Install Ingress Bare Metal
```
sudo kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.1/deploy/static/provider/baremetal/deploy.yaml

```
- Reference [bare Metal](https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal)

2. View Creation of Ingress (takes af few minutes)
```
sudo kubectl get pods -n ingress-nginx   -l app.kubernetes.io/name=ingress-nginx --watch
```
3. List Services with ingress-nginx namespace. Notice that there is no external IP
```
sudo kubectl get svc --namespace=ingress-nginx
```
4. Set the Ingress loadbalancers External IP Address - Replace IP Address
```
sudo kubectl patch svc ingress-nginx-controller -n ingress-nginx -p '{"spec": {"type": "LoadBalancer", "externalIPs":["54.244.4.114"]}}'
```
5. List Services with ingress-nginx namespace. Notice that now there is an external IP
```
sudo kubectl get svc --namespace=ingress-nginx
```

## Service Installs
1. Service -  apple

   sudo kubectl apply -f service1.yaml  
   or  
   sudo kubectl apply -f https://raw.githubusercontent.com/mallond/microk8s/main/service_apple.yaml  
   

List the Apple Service service
```
sudo kubectl get svc --namespace default
sudo kubectl get pods -n default

```

2. Service - banana

   sudo kubectl apply -f service2.yaml   
   or   
   sudo kubectl apply -f https://raw.githubusercontent.com/mallond/microk8s/main/service_banana.yaml   
   

List the Bannana Service service
```
sudo kubectl get svc --namespace default
sudo kubectl get pods -n default

```

3. InfluxDB 

Chart by Bitnami https://github.com/bitnami/charts/tree/master/bitnami/influxdb/#installing-the-chart
```
kubectl config view --raw > ~/.kube/config
helm repo add influxdata https://helm.influxdata.com/
helm upgrade --install my-release influxdata/influxdb2
```




## Ingress Services YAML
1. Ingress  
sudo kubectl apply -f service_ingress.yaml 
or  
sudo kubectl apply -f https://raw.githubusercontent.com/mallond/microk8s/main/service_ingress.yaml


> Note: Ingress implecetly supports TLS and the below annotations will force a redirect out of http to https.  
>
<img src="https://user-images.githubusercontent.com/993459/111708157-940b1b00-8802-11eb-81b3-f48a3a7c25b0.png" width="200" height="100"/>

```
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
```


## Test Ingress Controller

- sudo kubectl get svc --namespace=ingress-nginx
```
sudo kubectl get svc --namespace=ingress-nginx

NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller-admission   ClusterIP   10.152.183.29    <none>        443/TCP                      74s
ingress-nginx-controller             NodePort    10.152.183.142   <none>        80:30609/TCP,443:31953/TCP   73s
```
- curl https://10.152.183.142/apple -k
- curl https://10.152.183.142/bannana -k
- 
# Ingress Basic Authentication
- htpasswd -c auth myfoo 
- sudo kubectl create secret generic basic-auth --from-file=auth
- sudo kubectl get secret basic-auth -o yaml



## Add External IP
sudo kubectl patch svc ingress-nginx-controller -n ingress-nginx -p '{"spec": {"type": "LoadBalancer", "externalIPs":["3.15.237.254"]}}'
- netstat -ntlu
- netstat -na | grep :80
- sudo ufw allow 80
- sudo ufw enable


## Dashboard Proxy (hint to get the IP address)
```
microk8s dashboard-proxy
```

## Windows 11 and Azure Hacks
Escape the characters with \"
```
microk8s kubectl patch svc apple-service -p '{\"spec\": {\"type\": \"LoadBalancer\", \"externalIPs\":[\"172.23.100.158\"]}}'
```

Get to the VM
```
multipass shell microk8s-vm

```

Azure  
This will install networking plugs for the Azure VM   
Standard D8s v3 (8 vcpus, 32 GiB memory)  
```
apt-get install linux-azure
```

#### Test TLS
- curl https://10.152.183.142/banana -k
- Note: -k allows unsecured connections

# Kubectl useful memory aid
- sudo kubectl apply -f your.yaml  
- kubectl get svc  
- kubectl get svc --namespace=ingress-nginx
- kubectl get svs --all-namespaces
- kubectl get namespace
- kubectl cluster-info
- microk8s dashboard-proxy
- kubectl -n kube-system edit service kubernetes-dashboard    "Real time editing"
- curl http://localhost:32000/v2/_catalog?n=1                "Microk8s local registry"
- kubectl get deployments --all-namespacessudo microk8s dashboard-proxy "Dashboard check and token"
- kubectl describe svc influxdb -n influxdb
- kubectl logs -l app=banana
- kubectl cluster-info
- kubectl get services -n kube-system

# Helm useful memory aid
- helm list
- helm list --all 
- helm repo (list|add|update) 
- helm search  
- helm inspect <chart-name> 
- hem install --set a=b -f config.yaml <chart-name> -n <release-name> # --set take precedented, merge into -f 
- helm status <deployment-name> 
- helm delete <deployment-name> 
- helm inspect values <chart-name> 
- helm upgrade -f config.yaml <deployment-name> <chart-name> 
- helm rollback <deployment-name> <version> 
- helm create <chart-name> 
- helm package <chart-name> 
- helm lint <chart-name> 
- helm dep up <chart-name> # update dependency 
- helm get manifest <deployment-name> # prints out all of the Kubernetes resources that were uploaded to the server 
- helm install --debug --dry-run <deployment-name> # it will return the rendered template to you so you can see the output 

# Misc
ClusterIP: Exposes the service on a cluster-internal IP. Choosing this value makes the service only reachable from within the cluster. This is the default ServiceType

NodePort: Exposes the service on each Node’s IP at a static port (the NodePort). A ClusterIP service, to which the NodePort service will route, is automatically created. You’ll be able to contact the NodePort service, from outside the cluster, by requesting <NodeIP>:<NodePort>.

LoadBalancer: Exposes the service externally using a cloud provider’s load balancer. NodePort and ClusterIP services, to which the external load balancer will route, are automatically created.


