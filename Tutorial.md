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

