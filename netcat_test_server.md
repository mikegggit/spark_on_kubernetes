Netcat Test Server in Kubernetes
================================
How to use netcat in a k8 cluster for testing realtime endpoints, for example with Spark Streaming.

Background
----------
Testing spark streaming requires some kind of streaming endpoint accessible by the spark executors.  A simple TCP socket endpoint can be setup to support manually emitting text events to be consumed by spark jobs.

This is very useful for exploring the bundled spark streaming example apps.

Overview
--------
This installs a netcat listener in a k8s cluster and exposes it at well-known dns endpoint using a headless service.  A user can then attach to the container with stdin tty and enter test data.

Assumptions
-----------
* Docker Desktop is installed and configured
* kubectl is installed and configured

Steps
-----
#### Install netcat image in local repo.
See https://github.com/sostheim/nc-pod.

Requires a local docker repo.

#### Launch container in k8s and expose internally using headless service
From the nc-pod repo, replace nc-pod.yaml with the following:
```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nc-pod
  labels:
    name: nc-pod
    app: nc-pod
    version: 0.1.0
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nc-pod
  template:
    metadata:
      labels:
        app: nc-pod
        version: 0.1.0
        run: nc-pod
    spec:
      containers:
        - name: nc-pod
          image: localhost:5000/nc-pod:latest
          tty: true
          stdin: true
          env:
            - name: NC_CMD_ARGS
              value: "-vv -l -p 9999"
          ports:
          - containerPort: 9999
```
Note: tty:true and stdin:true are important to allow user entry into the server for transmission to connected clients.  

Add the following nc-headless-svc.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: nc-pod-svc
  labels:
    name: nc-pod-svc
    app: nc-pod
    version: 0.1.0
spec:
  # Optional - make the service headless by uncommenting the next line
  clusterIP: None
  selector:
    app: nc-pod
  ports:
  - name: nc-tcp-listener
    protocol: TCP
    port: 9999
```

Apply the templates to your k8s cluster
```
/var/tmp/nc-pod [chef:production] (master *%) $ k8 apply -f nc-pod.yaml
deployment.apps/nc-pod unchanged

/var/tmp/nc-pod [chef:production] (master *%) $ k8 apply -f nc-headless-svc.yaml
service/nc-pod-svc unchanged

/var/tmp/nc-pod [chef:production] (master *%) $ k8 get all
NAME                                   READY   STATUS    RESTARTS   AGE
pod/nc-pod-65dbc66c4c-g44xp            1/1     Running   1          100m

NAME                                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
service/kubernetes                             ClusterIP   10.96.0.1       <none>        443/TCP             2d21h
service/nc-pod-svc                             ClusterIP   None            <none>        9999/TCP            147m

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nc-pod   1/1     1            1           3h57m

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/nc-pod-65dbc66c4c   1         1         1       159m
```

Simple usage example
--------------------
#### Connect a simple client to the headless service
```
/var/tmp/nc-pod [chef:production] (master *%) $ kubectl run -i -t alps --image=alpine:latest --restart=Never
If you don't see a command prompt, try pressing enter.
/ #
```

#### Attach to the test server
```
~ [chef:production] $ k8 attach -it $(k8 get pod -l run=nc-pod -o name)
Defaulting container name to nc-pod.
Use 'kubectl describe pod/nc-pod-65dbc66c4c-g44xp -n default' to see all of the containers in this pod.
If you don't see a command prompt, try pressing enter.
```

#### Enter data into the server terminal
```
~ [chef:production] $ k8 attach -it $(k8 get pod -l run=nc-pod -o name)
Defaulting container name to nc-pod.
Use 'kubectl describe pod/nc-pod-65dbc66c4c-nqgb8 -n default' to see all of the containers in this pod.
If you don't see a command prompt, try pressing enter.
Connection from 10.1.1.4 35365 received!
one two
three four
one
two
three
four
```

Notice the server data is echoed in the client terminal.
```
/var/tmp/nc-pod [chef:production] (master *%) $ kubectl run -i -t alps --image=alpine:latest --restart=Never
If you don't see a command prompt, try pressing enter.
/ # nc -v nc-pod-svc 9999
nc-pod-svc (10.1.1.3:9999) open
one two
three four
one
two
three
four
```


Reference
---------
https://github.com/sostheim/nc-pod
https://docs.docker.com/registry/deploying/