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

## Setting up Postgres server on Node 1

The Docker Compose file required to set up the Postgres database is provided in this repository. First, navigate to the directory where the docker-compose-postgres.yaml file is located. Since Node 0 is the only node exposed to the public, you must transfer the file to Node 0 first and then forward it to Node 1.


```
scp -i <path to your private key> docker-compose-postgres.yaml cc@your_public_ip:/home/
```

This will move the YAML file to node 0. Then on node 0 run: 

```
scp  /home/docker-compose-postgres.yaml node-1:/home/
```

Finally run 
```
docker compose -f /home/docker-compose-postgres.yaml up -d
```

You can do a `docker ps` to check if the postgres server is succesfully up and running. 


This material is based upon work supported by the National Science Foundation under Grant No. 2230079.
