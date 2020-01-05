Running spark on a kubernetes cluster
=====================================
Simple guidelines for running a spark cluster on Docker Desktop (Kubernetes).  

Covers both spark core and spark streaming examples.  Spark streaming is more complex than the batch examples.

Goals
-----
* Document working examples.
* Focus on kubernetes more than spark
* Demonstrate using netcat for spark streaming poc

Assumptions
-----------
* Docker Desktop is installed and configured
* kubectl is installed and configured
* Spark 2.4.4 has been downloaded and installed to /usr/local/lib/spark
* You are using the built in spark folder structure when building docker images
* All required file dependencies are included in the image and _not_ mounted or made available to the containers otherwise.

Future ideas
------------
* Explore mounting resources via volume mount or similar

General notes
-------------
* You will need ~ 4GB memory allocated to the daemon to run two executors.

Uber jars
---------
See https://spark.apache.org/docs/latest/running-on-kubernetes.html.  

Notice that dependencies placed in the jars folder of the spark download file structure are automatically copied into the spark image and will be available to application code running within the cluster.  This can be useful if you don't want to bundle certain dependencies in the uber jar, or instead of an uber jar, simply want to include dependencies in the spark image.

It's also possible to use volume mounts to make certain resources available to the spark containers without bundling them inside an image.


Create a docker image
---------------------
The spark installation comes with a dockerfile.

To address issues mentioned here -> https://stackoverflow.com/questions/57643079/kubernetes-watchconnectionmanager-exec-failure-http-403, you may want to edit the dockerfile as follows:

```
COPY ${k8s_tests} /opt/spark/tests
COPY data /opt/spark/data

# ADDED by Mike G -> https://stackoverflow.com/questions/57643079/kubernetes-watchconnectionmanager-exec-failure-http-403
RUN rm /opt/spark/jars/kubernetes-client-4.1.2.jar
RUN rm /opt/spark/jars/kubernetes-model-4.1.2.jar
RUN rm /opt/spark/jars/kubernetes-model-common-4.1.2.jar

ADD https://repo1.maven.org/maven2/io/fabric8/kubernetes-client/4.4.2/kubernetes-client-4.4.2.jar /opt/spark/jars
ADD https://repo1.maven.org/maven2/io/fabric8/kubernetes-model/4.4.2/kubernetes-model-4.4.2.jar /opt/spark/jars
ADD https://repo1.maven.org/maven2/io/fabric8/kubernetes-model-common/4.4.2/kubernetes-model-common-4.4.2.jar /opt/spark/jars
# ADDED by Mike G -> https://stackoverflow.com/questions/57643079/kubernetes-watchconnectionmanager-exec-failure-http-403

```

Note: To run your own job, you will need to include the uber jar in the jars folder, and any other resources on which the job depends as well into one of the other directories, for example ${SPARK_HOME}/data.  

It is also possible to mount job dependencies separately from the image.


From the root spark directory => SPARK_HOME = /usr/local/lib/spark/...
```
docker build -t spark:latest -f kubernetes/dockerfiles/spark/Dockerfile .

...

Step 20/21 : WORKDIR /opt/spark/work-dir
 ---> Using cache
 ---> f0684d549e2f
Step 21/21 : ENTRYPOINT [ "/opt/entrypoint.sh" ]
 ---> Using cache
 ---> 742b90cc6ac9
Successfully built 742b90cc6ac9
Successfully tagged spark:latest
/usr/local/lib/spark [chef:production] $
```

Spark Core Example
------------------
See [Spark Batch](spark_batch.md).

Spark Streaming Example
---------------
See [Spark Streaming](spark_streaming.md).


Monitoring Job Progress
-----------------------
[chef:production] $ k8 port-forward `k8 get pod --field-selector=status.phase=Running -o name -l spark-role=driver` 4040:4040
Forwarding from 127.0.0.1:4040 -> 4040
Forwarding from [::1]:4040 -> 4040

Determining the K8s master url
------------------------------
```
/usr/local/lib/spark [chef:production] $ k8 cluster-info
Kubernetes master is running at https://kubernetes.docker.internal:6443
KubeDNS is running at https://kubernetes.docker.internal:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

General format for lauching a job
---------------------------------
Using an example streaming job that comes bundled w/ Spark...

Example submission script invocation:
```
./bin/spark-submit      
  --master k8s://https://kubernetes.docker.internal:6443      
  --deploy-mode cluster      
  --name spstreaming 
  --executor-memory 500M   
  --class org.apache.spark.examples.streaming.JavaCustomReceiver      
  --conf spark.executor.instances=2      
  --conf spark.kubernetes.container.image=spark:latest      
  local:///opt/spark/examples/jars/spark-examples_2.11-2.4.4.jar nc-pod-svc 9999
```

Discussion:
local:///opt/spark/examples/jars/spark-examples_2.11-2.4.4.jar references the path where the jar is installed inside the container filesystem.

Increasing the number of instances may require increasing the amount of cpu / memory resources availabe to the daemon.  You can also lower the amount of memory assigned to each executor.


Concerns
--------
### Using external content not bundled in an image
See https://spark.apache.org/docs/latest/running-on-kubernetes.html for exposing resources outside the image to running spark containers.

### Insufficient memory
Make sure to allocate enough memory to the docker daemon.  It may also help to change the number of executors / workers being requested.

### Passing args to spark main function
Add following the classname.

### Why not spark k8 operator
Not the point of this example.  Might be a good idea.


Reference
---------
https://spark.apache.org/docs/latest/running-on-kubernetes.html

