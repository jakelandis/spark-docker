# spark-docker-yarn
This repository contains Docker images for Apache Spark executed on Hadoop YARN. 
The purpose of them is to allow programmers to test Spark applications deployed on YARN easier. 
**It was not designed to be deployed in production environments**. The project was tested on Ubuntu 16. 

# Building the cluster
Unlike https://github.com/bartosz25/spark-docker/releases/tag/v1.0 version, this one uses `docker-compose` to create master and worker containers (nodes). It's executed with standard `docker-compose up` command and the number of workers is  defined with `--scale slave=X` property. 

But before calling it, 3 Docker images must be built with the help of `make`:
```
make build_base_image
make build_master_image
make build_slave_image
```

Now we can build a cluster with for intance 3 slaves, the following command must be used:
```
docker-compose up --scale slave=3
```  
# Add spark-master to /etc/hosts

On local machine (not sure why the docker compose entries dont work?):
```
127.0.0.1	localhost spark-master
```

# Spark and YARN UIs
Spark and YARN expose web UI used to track the execution of the applications:
* http://spark-master:8088 - YARN UI's address
* http://spark-master:18080 - Spark history UI's address

# Repository structure
* conf-master: stores master's configuration files are stored there
* conf-slave: stores slave's configuration files are stored there 
* master: contains master's Dockerfile
* slave: contains slave's Dockerfile
* shared-master: this repository is shared between master's Docker container (/home/sparker/shared) and host. 
* shared-slave: this repository is shared between slave Docker containers (/home/sparker/shared) and host

Shared repositories can be used to, for example, put the JAR executed with spark-submit inside.

# Tests
To verify that the cluster was correctly installed, launch _SparkPi_ example:
```
docker  exec -it spark-docker_master_1  bash

spark-submit --class co.elastic.sample.spark.Test1 --conf spark.es.nodes=elasticsearch  --master yarn --deploy-mode cluster --driver-memory 1g --executor-memory 1g --num-executors 3 --executor-cores 3 --jars ~/shared/*.jar

spark-submit --class co.elastic.sample.spark.Test1 --conf spark.es.nodes=elasticsearch  --master yarn --deploy-mode cluster --driver-memory 1g --executor-memory 1g --executor-cores 3 --jars ~/shared/*.jar

```

# Troubleshooting

```
docker  exec -it spark-docker_master_1  bash

spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode cluster --driver-memory 1g --executor-memory 1g --executor-cores 1 ~/spark-2.1.0-bin-hadoop2.7/examples/jars/spark-examples*.jar 1000

```

```
docker-compose up # only a single slave
docker  exec -it spark-docker_slave_1  bash
cd /home/sparker/hadoop-2.7.3/bin
./yarn logs -applicationId application_1607566675933_0001
```

Get local IP 
```
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' spark-docker_slave_1
```
Get local logs
```
docker logs -f elasticsearch 
```


## "Unhealthy Node local-dirs and log-dirs"
I encounter the issue when I had too few available disk space. It makes that the slave nodes are detected as unhealthy. You can fix that either by playing with the configuration 
https://stackoverflow.com/questions/29131449/why-does-hadoop-report-unhealthy-node-local-dirs-and-log-dirs-are-bad or simply by ensuring that you have enough free disk space.
