# Deploying the Agent

Now lets deploy the Datadog Agents. We have 3 options, you can use Helm, Datadog Operator or using a Daemonset. The main difference is that a Daemonset will make sure that an agent is installed in every node. In our case we just have 1 node.  

Helm let's you create a Chart which is a template for all of the Kubernetes resources you are interested in creating. The chart can then be installed chart into your cluster with customized data that can be provided via a values files or in the install command.

``` bash
# Check what repositories your Helm command currently has access too
helm repo list

# Add the charts repository so that your Helm can make use of charts
helm repo add stable https://charts.helm.sh/stable

# Add the datadog repository
helm repo add datadog https://helm.datadoghq.com

# If you already had access to these repos, you can update them to make sure you have the latest versions
helm repo update

# Update a specific repo
helm repo update <REPO_NAME>

# Remove a repo
helm repo remove <REPO_NAME>
```

Now that our repos are up-to-date, lets install the datadog agent onto our cluster. First lets get a clean values file from [github](https://github.com/DataDog/helm-charts/blob/main/charts/datadog/values.yaml) as this is needed to fill in the chart.

The easiest way to get this onto you computer is to use curl the url and add the output to a file called `values.yaml`.

``` bash
curl https://raw.githubusercontent.com/DataDog/helm-charts/master/charts/datadog/values.yaml > values.yaml
```

You can also go to the link and copy&paste the contents into a `values.yaml` file.

If you open the `values.yaml` file you will see that in under the `datadog` section, the `apiKey` key has a place holder. You have 2 options here, you can directly add your API KEY here or you can set it when installing the chart.  

``` bash
# Example command
helm install <RELEASE_NAME> -f values.yaml --set datadog.apiKey=<YOUR_API_KEY> datadog/datadog --set targetSystem=linux
```

Now that we have installed the chart, you should be able to see the pods that were deployed for you.

``` bash
# Get a list of all pods in your cluster
kubectl get pods
```

You should see 2 pods if you have made this release with a more up to date version of the helm chart. I'll explain in more details below:

The pod called `<RELEASE_NAME>-datadog-xxx` should be running and will have multiple conatiners running, the amount depends on the what checks were enabled. For example if APM or system probe was enabled you would have tha containers responsible for those features running in this pod.

The pod called `<RELEASE_NAME>-datadog-cluster-agent-xxx` is, as the name suggests the cluster agent pod.  

You might see customers with a 3rd pod called `<RELEASE_NAME>-kube-state-metrics-xxx`, then it is mostly because they are not using a more recent chart. This pod is responsible for collecting `kubernetes_state.*` metrics. However, in the newer version of our chart, this job is deligated to the cluster agent so this pod isn't required. The setting that enables this can be found in the `values.yaml` file under `datadog.kubeStateMetricsCore.enabled`, you can see that it is set to true.

Let's check if the kubelet and KSM checks are running as expected.

``` bash
# Run the agent status command
kubectl exec -it <AGENT_POD_NAME> agent status
```

Scroll to the kubelet section, if you see an error regarding:  

``` bash
Unable to detect the kubelet URL automaticallly...
```

To solve this you will need to update the `datadog.kubelet.tlsVerify` to `false` in the `values.yaml` and then redeploy and run the status command again.

``` status
helm upgrade <RELEASE_NAME> -f values.yaml --set datadog.apiKey=<YOUR_API_KEY> datadog/datadog -- set targetSystem=linux
```

If you forgot the release name you can run `helm list` to see your current releases.  
If you need to stop a release you can run `helm uninstall <RELEASE_NAME>`.

[< Creating a Kubernetes cluster on your machine](create-cluster.md) | [Beginning](../README.md) | [Configuring an Integration >](integration.md)