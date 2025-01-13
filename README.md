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




This material is based upon work supported by the National Science Foundation under Grant No. 2230079.