Running spark streaming job on a kubernetes cluster
===================================================
Simple example of running a spark streaming job on Docker Desktop (Kubernetes).

Working Example Steps - Word Count Example 
------------------------------------------
### Start the netcat container and headless service
See [Netcat Test Server in Kubernetes](netcat_test_server.md)

### Run the word-count example job
Submit the job.

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

Notice program arguments are specified after the jar path.  nc-pod-svc refers to the DNS entry created by the headless service, 9999 to the port at which the netcat server is listening.

### Attach to the netcat server and enter words.
```
~ [chef:production] $ k8 attach -it $(k8 get pod -l run=nc-pod -o name)
Defaulting container name to nc-pod.
Use 'kubectl describe pod/nc-pod-65dbc66c4c-nqgb8 -n default' to see all of the containers in this pod.
If you don't see a command prompt, try pressing enter.
one
two
three
four
one one one one
```


```
k8 logs $(k8 get pod -l spark-role=driver -o name) --follow

++ id -u
+ myuid=0
++ id -g
+ mygid=0
+ set +e
++ getent passwd 0
+ uidentry=root:x:0:0:root:/root:/bin/ash
+ set -e
+ '[' -z root:x:0:0:root:/root:/bin/ash ']'
+ SPARK_K8S_CMD=driver
+ case "$SPARK_K8S_CMD" in
+ shift 1
+ SPARK_CLASSPATH=':/opt/spark/jars/*'
+ env
+ grep SPARK_JAVA_OPT_
+ sort -t_ -k4 -n
+ sed 's/[^=]*=\(.*\)/\1/g'
+ readarray -t SPARK_EXECUTOR_JAVA_OPTS
+ '[' -n '' ']'
+ '[' -n '' ']'
+ PYSPARK_ARGS=
+ '[' -n '' ']'
+ R_ARGS=
+ '[' -n '' ']'
+ '[' '' == 2 ']'
+ '[' '' == 3 ']'
+ case "$SPARK_K8S_CMD" in
+ CMD=("$SPARK_HOME/bin/spark-submit" --conf "spark.driver.bindAddress=$SPARK_DRIVER_BIND_ADDRESS" --deploy-mode client "$@")
+ exec /sbin/tini -s -- /opt/spark/bin/spark-submit --conf spark.driver.bindAddress=10.1.1.5 --deploy-mode client --properties-file /opt/spark/conf/spark.properties --class org.apache.spark.examples.streaming.JavaCustomReceiver spark-internal nc-pod-svc 9999
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/opt/spark/jars/spark-soup-receiver-1.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/opt/spark/jars/slf4j-log4j12-1.7.16.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
[                          main] NativeCodeLoader               WARN  Unable to load native-hadoop library for your platform... using builtin-java classes where applicable

-------------------------------------------
Time: 1578258296000 ms
-------------------------------------------

-------------------------------------------
Time: 1578258297000 ms
-------------------------------------------
(one,1)

-------------------------------------------
Time: 1578258298000 ms
-------------------------------------------
(two,1)

-------------------------------------------
Time: 1578258300000 ms
-------------------------------------------
(three,1)

-------------------------------------------
Time: 1578258301000 ms
-------------------------------------------
(four,1)

-------------------------------------------
Time: 1578258306000 ms
-------------------------------------------
(one,4)
```



Reference
---------
https://spark.apache.org/docs/latest/running-on-kubernetes.html

