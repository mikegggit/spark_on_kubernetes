Running spark batch job on a kubernetes cluster
===============================================
Simple example of running a spark batch job on Docker Desktop (Kubernetes).  

Working Example Steps - SparkPi Example 
---------------------------------------
### Run the spark pi example job
Submit the job.

```
/usr/local/lib/spark [chef:production] $ ./bin/spark-submit \
>     --master k8s://https://kubernetes.docker.internal:6443 \
>     --deploy-mode cluster \
>     --name spark-pi \
>     --class org.apache.spark.examples.SparkPi \
>     --conf spark.executor.instances=2 \
>     --conf spark.kubernetes.container.image=spark:latest \
>     local:///opt/spark/examples/jars/spark-examples_2.11-2.4.4.jar
log4j:WARN No appenders could be found for logger (io.fabric8.kubernetes.client.Config).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
20/01/03 11:04:24 INFO LoggingPodStatusWatcherImpl: State changed, new state:
   pod name: spark-pi-1578067462467-driver
   namespace: default
   labels: spark-app-selector -> spark-7142808b0bdc4bd98aacfa6d704d9764, spark-role -> driver
   pod uid: 44d4cc08-bc76-4a3b-8d69-f033522b2c55
   creation time: 2020-01-03T16:04:24Z
   service account name: default
   volumes: spark-local-dir-1, spark-conf-volume, default-token-r9686
   node name: N/A
   start time: N/A
   container images: N/A
   phase: Pending
   status: []
20/01/03 11:04:24 INFO LoggingPodStatusWatcherImpl: State changed, new state:
   pod name: spark-pi-1578067462467-driver
   namespace: default
   labels: spark-app-selector -> spark-7142808b0bdc4bd98aacfa6d704d9764, spark-role -> driver
   pod uid: 44d4cc08-bc76-4a3b-8d69-f033522b2c55
   creation time: 2020-01-03T16:04:24Z
   service account name: default
   volumes: spark-local-dir-1, spark-conf-volume, default-token-r9686
   node name: docker-desktop
   start time: N/A
   container images: N/A
   phase: Pending
   status: []
20/01/03 11:04:24 INFO LoggingPodStatusWatcherImpl: State changed, new state:
   pod name: spark-pi-1578067462467-driver
   namespace: default
   labels: spark-app-selector -> spark-7142808b0bdc4bd98aacfa6d704d9764, spark-role -> driver
   pod uid: 44d4cc08-bc76-4a3b-8d69-f033522b2c55
   creation time: 2020-01-03T16:04:24Z
   service account name: default
   volumes: spark-local-dir-1, spark-conf-volume, default-token-r9686
   node name: docker-desktop
   start time: 2020-01-03T16:04:24Z
   container images: spark:latest
   phase: Pending
   status: [ContainerStatus(containerID=null, image=spark:latest, imageID=, lastState=ContainerState(running=null, terminated=null, waiting=null, additionalProperties={}), name=spark-kubernetes-driver, ready=false, restartCount=0, state=ContainerState(running=null, terminated=null, waiting=ContainerStateWaiting(message=null, reason=ContainerCreating, additionalProperties={}), additionalProperties={}), additionalProperties={})]
20/01/03 11:04:24 INFO Client: Waiting for application spark-pi to finish...
20/01/03 11:04:25 INFO LoggingPodStatusWatcherImpl: State changed, new state:
   pod name: spark-pi-1578067462467-driver
   namespace: default
   labels: spark-app-selector -> spark-7142808b0bdc4bd98aacfa6d704d9764, spark-role -> driver
   pod uid: 44d4cc08-bc76-4a3b-8d69-f033522b2c55
   creation time: 2020-01-03T16:04:24Z
   service account name: default
   volumes: spark-local-dir-1, spark-conf-volume, default-token-r9686
   node name: docker-desktop
   start time: 2020-01-03T16:04:24Z
   container images: spark:latest
   phase: Running
   status: [ContainerStatus(containerID=docker://672f2deb529568b203fb6176c7b3412e39da4e4669b075325d6b630c5fe759a1, image=spark:latest, imageID=docker://sha256:742b90cc6ac9ffab3ce6c0b31cd31a96a0599f45c6599f0d595f563c3bf9c897, lastState=ContainerState(running=null, terminated=null, waiting=null, additionalProperties={}), name=spark-kubernetes-driver, ready=true, restartCount=0, state=ContainerState(running=ContainerStateRunning(startedAt=2020-01-03T16:04:25Z, additionalProperties={}), terminated=null, waiting=null, additionalProperties={}), additionalProperties={})]
20/01/03 11:05:03 INFO LoggingPodStatusWatcherImpl: State changed, new state:
   pod name: spark-pi-1578067462467-driver
   namespace: default
   labels: spark-app-selector -> spark-7142808b0bdc4bd98aacfa6d704d9764, spark-role -> driver
   pod uid: 44d4cc08-bc76-4a3b-8d69-f033522b2c55
   creation time: 2020-01-03T16:04:24Z
   service account name: default
   volumes: spark-local-dir-1, spark-conf-volume, default-token-r9686
   node name: docker-desktop
   start time: 2020-01-03T16:04:24Z
   container images: spark:latest
   phase: Succeeded
   status: [ContainerStatus(containerID=docker://672f2deb529568b203fb6176c7b3412e39da4e4669b075325d6b630c5fe759a1, image=spark:latest, imageID=docker://sha256:742b90cc6ac9ffab3ce6c0b31cd31a96a0599f45c6599f0d595f563c3bf9c897, lastState=ContainerState(running=null, terminated=null, waiting=null, additionalProperties={}), name=spark-kubernetes-driver, ready=false, restartCount=0, state=ContainerState(running=null, terminated=ContainerStateTerminated(containerID=docker://672f2deb529568b203fb6176c7b3412e39da4e4669b075325d6b630c5fe759a1, exitCode=0, finishedAt=2020-01-03T16:05:02Z, message=null, reason=Completed, signal=null, startedAt=2020-01-03T16:04:25Z, additionalProperties={}), waiting=null, additionalProperties={}), additionalProperties={})]
20/01/03 11:05:03 INFO LoggingPodStatusWatcherImpl: Container final statuses:


   Container name: spark-kubernetes-driver
   Container image: spark:latest
   Container state: Terminated
   Exit code: 0
20/01/03 11:05:03 INFO Client: Application spark-pi finished.
20/01/03 11:05:03 INFO ShutdownHookManager: Shutdown hook called
20/01/03 11:05:03 INFO ShutdownHookManager: Deleting directory /private/var/folders/r4/ys__xxm97vn_hlt_9m0nfj2h0000gn/T/spark-44ba77dc-74ef-4587-8e5c-addc21504fa1
```



