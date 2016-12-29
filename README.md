# Kubernetes on Local with Minikube Tutorial

## Install
You need to install in your machine :
- Docker (https://docs.docker.com/engine/installation/)
- Minikube (https://github.com/kubernetes/minikube/releases) And the requirement of minikube (https://github.com/kubernetes/minikube#installation)
- Kubernetes command-line tool (http://kubernetes.io/docs/user-guide/prereqs/)

## Let's Play

### Starting ...

First start minikube that will run in local kubernetes (instead have to use google cloud or AWS )
```bash
minikube start
```

Check that Kubernetes are running well 
```bash
kubectl cluster-info
```

### First example :
For your first example you can follow the one given by minikube itself : https://github.com/kubernetes/minikube#quickstart

### Second example :
We will do more fun things that this first example.
Once minikube is started you can do everything kubernetes allow (there are a lot of tutorial in internet) with some differences that we will see throught an example :

Let's begin by creating 2 replication of an nginx docker load balanced on Kubernetes.

```bash
kubectl run test-nginx --image=nginx --replicas=2 --port=80 --expose --service-overrides='{ "spec": { "type": "LoadBalancer" } }'
```
You can see your 2 replicas some seconds later "running" by typing 

```bash
kubectl get pod
```

Once it's running you can see their internal IP 
```bash
kubectl get services
```

You can't see their external IP, because it's running in a minikube
To Test you can find the ip by typing : 
```bash
minikube service test-nginx --url 
```
![ngnix_screen](/nginx_screen1.png)

or if you are using linux :
```bash
curl $(minikube service test-nginx --url)
```

#### Playing with replicas

Let's delete one of our pod, and see how kubernetes behave :
So get the name of one of your pods typing :
```bash
kubectl get pod
```

Once you have the name :

```bash
kubectl delete pod [name]
```

And right after you will see kubernetes creating an other container to continue having 2 replicas :
```bash
kubectl get pod
```

#### Entering in a container

check the name of your pods
```bash
kubectl get pod
```

Execute the following command with the name of your first pod.
```bash
kubectl exec -ti test-nginx [name] --bash
```

You will have the prompt commnd inside of the nginx container.
Type : 
```bash
cd /usr/share/nginx.html
rm index.html & echo "<html><body><h1>Hello World</h1></body></html>" > index.html
exit
```

reload the page


#### Cleaning

Don't forget to clean your room :

```bash
kubectl delete deployment test-nginx
kubectl delete service test-nginx
minikube stop
```

### Third Example

We will try to dockerize our own code.
This comes from the excellent tutorial http://kubernetes.io/docs/hellonode/, but applied to minikube

Create in a directory 2 files : 
- server.js
- dockerfile


Edit server.js with :
```js
const http = require('http');
const handleRequest = (request, response) => {
  console.log('Received request for URL: ' + request.url);
  response.writeHead(200);
  response.end('Hello World!');
};
const www = http.createServer(handleRequest);
www.listen(8080);
```

Edit the dockerfile with : 
```bash
FROM node:4.5
EXPOSE 8080
COPY server.js .
CMD node server.js
```

Now we will build your docker file inside minikube for that on your console type :
```bash
eval $(minikube docker-env)
```

if you do a docker images you will see that in this console you are not anymore inside your own docker, but inside the minikube one.

So build your dockerfile :
```bash
docker build -t hellonode:v1 .
```

Now we will mount this image :
```bash
kubectl run test-node --image=hellonode:v1 --port=8080
```

you can see it deployed :
```bash
kubectl get deployments
```

you can see it running :
```bash
kubectl get pods
```
Once it is running, we will allow external traffic :

```bash
kubectl expose deployment test-node --type="LoadBalancer"
```

check :
```bash
kubectl get services
```

we can now test : 

```bash
curl -i $(minikube service test-node --url)
HTTP/1.1 200 OK
Date: Fri, 9 Dec 2016 14:57:19 GMT
Connection: keep-alive
Transfer-Encoding: chunked

Hello World!
```

#### Scale

With Kubernetes it's easy to scale, we only need one command line :

```bash
kubectl scale deployment test-node --replicas=4
```

And in order to check :
```bash
kubectl get deployments
```
```bash
kubectl get pods
```

## Links :
- https://github.com/kubernetes/minikube
- http://kubernetes.io/docs/user-guide/kubectl-cheatsheet/
- http://kubernetes.io/docs/hellonode/
