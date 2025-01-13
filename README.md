# testbeds-MLFlow

## ML Flow 

This repository serves as a demonstartion for a complete ML development workflow using MLflow, deployed on a Kubernetes cluster within Chameleon Cloud. Our setup follows a 3 node system where as shown in the image below. Each of the 3 remote hosts serves as the tracking server, model registry, and an SQL store (to store params, metrics, runs...etc) 

![MLFlow](https://mlflow.org/docs/latest/_images/scenario_5.png)

The tracking server is exposed to the public, serving as the interface for our complete workflow. This setup allows the members of a team to train models independently, while keeping track of all the experiment parameters and results in one place.

## Setting up the VMs on Chameleon Cloud

This experiment assumes you have already completed  [Hello, Chameleon](https://teaching-on-testbeds.github.io/blog/hello-chameleon), so you have set up your Chameleon account, created and added keys to your account, and joined an active project on Chameleon.

Running the  `reserve_chameleon.ipynb` notebook intiatialize the 3 node setup for you to run your experiments. From the [Chameleon website](https://chameleoncloud.org/), click on "Experiment > Jupyter Interface" in the menu. You may be prompted to log in. You will then upload the `reserve_chameleon.ipynb` and run all cells to launch the VMs

If your notebook runs without any issues you will see an output in the "Network Topolgy" section that looks like this:

![Network Topology](./images/nettop.png)


* Node 0 in our setup is the tracking server that is exposed to the public
* Node 1 and Node 2 will run the postgres DB and the MiniIO artifact stores repectively

## Set up Postgres server on Node 1

The Docker Compose file required to set up the Postgres database is provided in this repository. First, navigate to the directory where the docker-compose-postgres.yaml file is located. Since Node 0 is the only node exposed to the public, you must transfer the file to Node 0 first and then forward it to Node 1.


```
scp -i <path to your private key> docker-compose-postgres.yaml cc@your_public_ip:/home/
```

This will move the YAML file to node 0. Then on node 0 run: 

```
scp  /home/docker-compose-postgres.yaml node-1:/home/
```

Finally ssh onto node-1 from node-0 and run:
```
docker compose -f /home/docker-compose-postgres.yaml up -d
```

You can do a `docker ps` to check if the postgres server is succesfully up and running. 

## Set up MinIO artifact store on Node 2

On node2 we will set up MinIO, an S3-compatible object storage, as an artifact store so that you don’t need to have AWS account to run this experiment. We will move the docker-compose files to node1 in a similar way we did for node 1.  

Navigate to the directory where the docker-compose-postgres.yaml file is located and run

```
scp -i <path to your private key> docker-compose-minio.yaml cc@your_public_ip:/home/
```

Then on node-0 run:

```
scp  /home/docker-compose-minio.yaml node-2:/home/
```

Finally, on node-2 run:
```
docker compose -f /home/docker-compose-minio.yaml up -d
```

This will create an artifact store and also initialize and empty bucket named `mlflowbucket` on node2 

## Set up the MLFlow tracking server - Node 0

We now have the artifact store running on node-2 and the Postgres server running on node-1. All we have to now do is to launch the tracking server on node-0. For the tracking server to access remote storage, it needs to be configured with the necessary credentials.

```
export MLFLOW_S3_ENDPOINT_URL=http://192.168.1.12:5000 
export AWS_ACCESS_KEY_ID=minio_user
export AWS_SECRET_ACCESS_KEY=minio_password
```

Note that these credentials are the ones specified in `docker-compose-minio.yaml`. If you used different credentials in your YAML, you will have to change the credentials accordingly. 

Finally, to launch the tracking server you will run: 

```
mlflow server \
  --backend-store-uri postgresql://user:password@192.168.1.11:5432/mlflowdb \
  --artifacts-destination s3://mlflowbucket \
  --host 0.0.0.0 \
  --port 8000
```

If you used different credntials in your docker-compose postgres, you will have to replace the command with the appropriate credentials. Your tracking server is now running and is accissible at `http://<public IP of node 0>:8000`.


This material is based upon work supported by the National Science Foundation under Grant No. 2230079.
