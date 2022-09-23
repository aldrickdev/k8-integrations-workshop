# Creating a Kubernetes cluster on your machine

Like I mentioned before, `minikube` allows us to create a local kubernetes cluster. So lets create one, run: `minikube start`.

``` bash
# Starting up your local cluster, with default name
minikube start

# Starting a cluster with a custom name
minikube start -p <CUSTOM_NAME>
```

Common commands minikube commands you might run:

``` bash
# To see the cluster minikube has created
minikube profile list

# Stop a running cluster (default)
minikube stop
# Stop a running cluster (custom name)
minikube stop -p <CUSTOM_NAME>

# Check the status of you cluster
minikube status
minkube status -p <CUSTOM_NAME>

# Delete a cluster
minikube delete
minikube delete -p <CUSTOM_NAME>
```

[< Tools](tools.md) | [Beginning](../README.md) | [Deploying the Agent >](deploy-agent.md)