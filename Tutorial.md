### K8s Installaion
 Please check [README.md](./README.md)

### deployment
It's a logical representation for one or many pods 

```sh
 # (kubmaster)
 kubectl create deployment mynginx --image nginx
```
to show deploy information

```sh
 # (kubmaster)
kubectl describe deploy mynginx
```

### Services
An abstract way to expose an application running on a set of Pods as a network service

create a serice to access the pod

```sh
kubectl create service nodeport mynginx --tcp=8080:80
```
show kub services and find the port that K8s expose the nginx pod

```sh
kubectl get service
```

to access to the pod from external browser:

http://192.168.56.102:<PORT - like 31363 ..>/

### scaling
first we will continue with the same previous example (nginx pod) 
we update the /usr/share/nginx/html/index.html inside the pod

```sh
kubectl exec -it mynginx-<id> /bin/bash

# inside the container
echo "instance 1" >  /usr/share/nginx/html/index.html 
```
to scale :

```sh
kubectl scale deploy mynginx --replicas=2
```
we do the same for the new pod (update index.html of ngnix )

```sh
kubectl exec -it mynginx-<id> /bin/bash

# inside the container
echo "instance 2" >  /usr/share/nginx/html/index.html 
```
now the load balacing is up :) 

we can go back to one only replicas:

```sh
kubectl scale deploy mynginx --replicas=1
```
to enable to autoscale:

```sh
kubectl autoscale deployment mynginx --min 1 --max 5
```

### Debugging
to watch modification , example:

```sh
watch kubectl get pods
```

list nodes and their status

```sh
kubectl describe nodes kubmaster
```
get components
```sh
kubectl get componentstatuses  # or " kubectl get cs " 
kubectl get daemonsets -n kube-system
kubectl get all
```
useful options : \
 **-o wide**  => more details \
 **-n kube-system** => display system info (namespaces)

to show logs:

```sh
kubectl logs <POD>
```
events:
```sh
kubectl get events
kubectl describe pods <POD>
```
### Export Resources

To generate configurations files via CLI , from an existant deployment:

```sh
kubectl get deploy mynginx -o yaml > mydeploy.yml
```
Edit the file and delete all the status section .
Now we can delete the current deployment
```sh
kubectl delete deployments mynginx
```
we test the deplyment creation from the yaml file:
```sh
kubectl apply -f mydeploy.yml
```
**Note :** we can also generate a config file (yaml) for a service 

