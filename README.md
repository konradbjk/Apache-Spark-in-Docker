# Requirements
* about 10 GB free disc space, 4 cores, 8 GB of RAM
* docker application installed [Community edition is free](https://docs.docker.com/compose/install/)
* docker compose - it might not be installed by default
* installed [git](https://gist.github.com/derhuerst/1b15ff4652a867391f03)


In the next step run:
```bash
# fetch docker image
docker pull dimajix/jupyter-spark

# fetch repo
git clone https://github.com/konradbjk/Apache-Spark-in-Docker.git

# Get into repo
cd Apache-Spark-in-Docker
```

# Disclaimer
This is docker configuration for dimajix/jupyter-spark image. I am not the person who develops it. I improved this configuration for better use of Apache Spark.

It was used on Workshop with [Grzegorz Gawron](https://github.com/gregaw/) about Apache Spark an docker containers. His presentation is available [here](https://github.com/gregaw/workshop-break-spark/blob/master/Let's%20Break%20Apache%20Spark%20Workshop.pdf)
# Usefull comands
```bash
# show master logs
docker exec -it jupyter-spark-master tail -f /opt/spark/logs/spark--org.apache.spark.deploy.master.Master-1-spark-master.out -n 1000

# show slave1 logs
docker exec -it jupyter-spark-slave-1 tail -f /opt/spark/logs/spark--org.apache.spark.deploy.worker.Worker-1-spark-slave-1.out -n 1000

# show slave2 logs
docker exec -it jupyter-spark-slave-2 tail -f /opt/spark/logs/spark--org.apache.spark.deploy.worker.Worker-2-spark-slave-2.out -n 1000

# run bash on jupyter-notebook
docker exec -it jupyter-notebook bash
cd /opt/spark-2.2.0-bin-without-hadoop/

# run spark-submit
bin/spark-submit --executor-memory=1G --conf "spark.driver.memory=1G" --conf "spark.cores.max=10" --conf "spark.executor.cores=1" --master spark://spark-master:7077 examples/src/main/python/pi.py

# bring the whole system up
docker-compose up

# bring the whole system down
docker-compose down

# show running docker containers
docker ps

# show configuration of a container
docker inspect jupyter-notebook

# stop one of the 'boxes'
docker stop jupyter-spark-master

# start one of the 'boxes'
docker start jupyter-spark-master

# create a required network to connect the containers
docker network create dimajix

```
# Jupyter Spark Docker Container

This Docker image contains a Jupyter notebook with a PySpark kernel. Per default, the kernel runs in Spark 'local'
mode, which does not require any cluster. But the Docker image also supports setting up a Spark standalone cluster
which can be accessed from the Notebook.

The easiest way to run the Jupyter notebook is to run:

```bash
    docker run -p 8888:8888 dimajix/jupyter-spark
```

Then when the container is running, point your web browser to `http://localhost:8888`, where the Jupyter notebook
server is running

You can also specify your AWS credentials for accessing data inside S3 via environment variables
```
    docker run \
        -p 8888:8888 \
        -e AWS_ACCESS_KEY_ID=your_aws_key \
        -e AWS_SECRET_ACCESS_KEY=your_aws_secret \
        dimajix/jupyter-spark
```

# Configuration

There are some configuration options which can be changed by setting environment variables for your Docker container.
Details to all the options are listed below.

## Jupyter Configuration

There are only two Jupyter specific configuration properties:
```
    JUPYTER_PORT=8888
    JUPYTER_DIR=/mnt/notebooks
```
## Spark Kernel Configuration

The Jupyter Notebook contains a special PySpark kernel, which also has some configuration options related to Spark
itself. Unfortunately you cannot change these settings on a per-notebook basis, but at least you can change these
settings per Docker container.
```
    SPARK_MASTER=local[*]
    SPARK_DRIVER_MEMORY=2G
    SPARK_EXECUTOR_MEMORY=4G
    SPARK_EXECUTOR_CORES=4
    SPARK_NUM_EXECUTORS=1    
```


## S3 properties

Since many users want to access data stored on AWS S3, it is also possible to specify AWS credentials and general
settings.
```
    S3_PROXY_HOST=
    S3_PROXY_PORT=
    S3_PROXY_USE_HTTPS=false
    S3_ENDPOINT=s3.amazonaws.com
    S3_ENDPOINT_HTTP_PORT=80
    S3_ENDPOINT_HTTPS_PORT=443

    AWS_ACCESS_KEY_ID=
    AWS_SECRET_ACCESS_KEY=
```


# Spark Cluster Configuration

Aside from the Jupyter kernel / driver side there are some more Spark related configuration properties, which are used
to setup and connect to the Spark cluster. Note that all worker nodes also require the same Python installation as on
the notebook server, so essentially the only deployment mode currently supported is Spark Standalone cluster using the
same Docker image for both the Spark master and all Spark worker nodes.

The following settings configure Spark master and all workers.
```
    SPARK_MASTER_HOST=spark-master
    SPARK_MASTER_PORT=7077

    SPARK_WEBUI_PORT=9090
    SPARK_WORKER_CORES=4
    SPARK_WORKER_MEMORY=8G
    SPARK_LOCAL_DIRS=/tmp/spark-local
    SPARK_WORKER_DIR=/tmp/spark-worker
```
## Hadoop Properties

It is possible to access Hadoop resources (in HDFS) from Spark.
```
    HDFS_NAMENODE_HOSTNAME=hadoop-namenode
    HDFS_NAMENODE_PORT=8020
    HDFS_DEFAULT_FS=${HDFS_DEFAULT_FS=hdfs://$HDFS_NAMENODE_HOSTNAME:$HDFS_NAMENODE_PORT}
    HDFS_REPLICATION_FACTOR=2
```
## Running a Spark Standalone Cluster

The container already contains all components for running a Spark standalone cluster. This can be achieved by using the
two commands
* master
* slave

The docker-compose file contains an example of a complete Spark standalone cluster with a Jupyter Notebook as the
frontend.
