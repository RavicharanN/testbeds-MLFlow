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
* Node 1 and Node 2 will run the MiniIO artifact stores and PostgresDB repectively

## Set up MinIO artifact store on Node 1

On node1 we will set up MinIO, an S3-compatible object storage, as an artifact store so that you don’t need to have AWS account to run this experiment. The `reserve_chameleon` note book should have already created a docker compose file to start the MinIO service on Node1. 

To get the service up, you will SSH into node-1 (via node-0) and run:

```
docker compose -f /home/docker-compose-minio.yaml up -d
```

You can do a `docker ps` to check if the postgres server is succesfully up and running. 

## Set up Postgres server on Node 2

On node2 we will set up the postgres server that will be used as a database that logs the parameters of all the experiments we run. The `reserve_chameleon` note book should have already created a docker compose file to start the Postgres service on Node1. 

To get the service up, you will SSH into node-2 (via node-0) and run:

```
docker compose -f /home/docker-compose-postgres.yaml up -d
```

You can run a `docker ps` on both node-0 and node-1 to check if the services are successfully up and running.

## Set up the MLFlow tracking server - Node 0

We now have the artifact store running on Node 2 and the Postgres server running on Node 1. The final step is to launch the tracking server on Node 0. For the tracking server to access remote storage, it needs to be configured with the appropriate credentials.

### Step 1: Set Environment Variables for Remote Storage

Run the following commands on Node 0:

```
export MLFLOW_S3_ENDPOINT_URL=http://192.168.1.11:5000 
export AWS_ACCESS_KEY_ID=minio_user
export AWS_SECRET_ACCESS_KEY=minio_password
```
> **Note:** These credentials are defined in docker-compose-minio.yaml. If you used different credentials in your YAML, update these commands accordingly.

### Step 2: Launch the Tracking Server

Run the following command on Node 0 to start the tracking server:

```
mlflow server \
  --backend-store-uri postgresql://user:password@192.168.1.12:5432/mlflowdb \
  --artifacts-destination s3://mlflowbucket \
  --host 0.0.0.0 \
  --port 8000
```
> **Note:** If you used different credentials for the Postgres database in your docker-compose-postgres.yaml, update the --backend-store-uri option with the correct values.

### Access Tracking Server

Your tracking server is now running and accessible at:

```
http://<public IP of Node 0>:8000
```


This material is based upon work supported by the National Science Foundation under Grant No. 2230079.
