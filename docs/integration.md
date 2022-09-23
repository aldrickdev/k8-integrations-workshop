# Configuring an Integration

There are many ways to configure an integration, You have autodiscovery which is when you configure the integration from the side of the pod using annotations, creating your own `conf.yaml` file for the integration and mounting it into the agent, using ConfigMap, using a key-value store and lastly adding the configuration directly to the `values.yaml`.

## Annotations

Let's go over configuring a redis integration using annotations. Create a `redis.yaml` and insert the follow configuration.

``` yaml 
apiVersion: v1
kind: Pod
metadata:
  name: redis
  labels:
    name: redis
spec:
  containers:
    - name: redis
      image: redis
      ports:
        - containerPort: 6379
```

Now lets apply this configuration.

``` bash
kubectl apply -f redis.yaml
```

If you run `kubectl get pods` you should see that now a `redis` pod with 1 container has been deployed. Lets run the agent status command just to see if we see a redis check. Here you will notice that yes we have a `redisdb` check running but how? We didnt configure an integration. If you look at the `Configuration Source` you will see that it is using a file called `auto_conf.yaml`. The Datadog agent can discover running containers and see if a configuration for them was set. If no configuration was created then it will see if an `auto_conf.yaml` file exist for the integration. You can find a list of all the integrations we have auto configs for [HERE](https://docs.datadoghq.com/agent/guide/auto_conf/#pagetitle). Lets say for some reason you want to deploy an instance of soemthing that there is an auto config for and you don't want the agent to track it, you can add the `DD_IGNORE_AUTOCONF` configuration to the `datadog.env` section of the `values.yaml` file. Below is the example if you wanted the agent to ignore the `redisdb` integration.

``` yaml
datadog:
  ...
  env:
    - name: DD_IGNORE_AUTOCONF
      value: "redisdb"
```

You can then upgrade the release with the new changes:

``` bash 8.
helm upgrade <RELEASE_NAME> -f values.yaml --set datadog.apiKey=<YOUR_API_KEY> datadog/datadog --set targetSystem=linux
```

Now if you were to run the agent check again, you shouldn't see the `redisdb` check anymore. Now lets configure the integration with using annotations. Update your `redis.yaml` file to the following:

This is the how you would define the annotations for autodiscovery v1.

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
  annotations:
    ad.datadoghq.com/redis.check_names: '["redisdb"]'
    ad.datadoghq.com/redis.init_configs: '[{}]'
    ad.datadoghq.com/redis.instances: |
      [
        {
          "host": "%%host%%",
          "port":"6379"
        }
      ]      
  labels:
    name: redis
spec:
  containers:
    - name: redis
      image: redis
      ports:
        - containerPort: 6379
```

Starting with agent version 7.36, version 2 of the autodiscovery annotations was added to simplify the configuration process.

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
  annotations:
    ad.datadoghq.com/redis.checks: |
      {
        "redisdb": {
          "instances": [
            {
              "host": "%%host%%",
              "port": "6379",
            }
          ]
        }
      }
  labels:
    name: redis
spec:
  containers:
    - name: redis
      image: redis
      ports:
        - containerPort: 6379
```

Once you have made the changes you can redeploy the `redis` pod and when you run the agent status command you should see that you now have the redis check running again but the `Configuration Source` now shows the container ID of your `redis` container showing that the agent is actually using our annotations as the integrations configurations. Note you do not need to set the ignore environment variable to use annotations as the configure as this would take precident over the auto configure anyways.

## ConfigMap

ConfigMaps are a Kubernetes resource you can create that is essentially kubernetes way of creating configuration files for your applications. In this case, we are using this Kubernetes specific feature and using it to define our integration configuration.

Lets delete the deployed redis pod so we can start from scratch.

``` bash
kubectl delete pod redis
```

Create a file called `redisConfigMap.yaml` and insert the configurations below:

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: redisdb-config-map
  namespace: default
data:
  redisdb-config: |-
    ad_identifiers:
      - redis
    init_config:
    instances:
      - host: "%%host%%"
        port: "6379" 
```

Some things to watch out for when creating these is that the structure of the configuration name in the `data` section is `<NAME_OF_CHECK>-config` and that in the `data` section, the `ad_identifiers` matches the container short image name, in this case `redis`.

Now we can deploy the ConfigMap.

``` bash
kubectl apply -f redisConfigMap.yaml
```

If you run `kubectl get configmap` you will see a list of all of the configurations, you should be able to see our redis configmap with the name we put as `metadata.name`.

In order for the agent to use this configuration, we need to tell the agent to mount this as a volume, update the `agents.volumes` and the `agent.volumeMounts` to match the configurations below:

``` yaml
agents:
  ...
  volumes:
    - name: redisdb-config-map
      configMap:
        name: redisdb-config-map
        items:
          - key: redisdb-config
            path: conf.yaml
  
  volumeMounts:
    - name: redisdb-config-map
      mountPath: /etc/datadog-agent/conf.d/redisdb.d
```

Once these changes have been made, you can redploy the agent.

``` bash
helm upgrade <RELEASE_NAME> -f values.yaml --set datadog.apiKey=<YOUR_API_KEY> datadog/datadog --set targetSystem=linux
```

Now before we redeploy the redis pod, lets remove the annotations so that we know that the configration isn't comming from autodiscovery. The `redis.yaml` file should look like:

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: redisdb-config-map
  namespace: default
data:
  redisdb-config: |-
    ad_identifiers:
      - redis
    init_config:
    instances:
      - host: "%%host%%"
        port: "6379" 
```

Now deploy the `redis` pod and then lets run the agent check.

``` bash
kubectl apply -f redis.yaml
```

``` bash
kubectl exec -it <AGENT_POD> agent status
```

You should be able to see the `redisdb` check. Looking at the `Configuration Source` you can see that it now says that its getting its configuration for a file called `conf.yaml` in a directory `/etc/datadog-agent/conf.d/redisdb.d/` as defined by the volume and volumeMount we added to the `values.yaml`. Just for fun lets see whats inside that file.

``` bash
kubectl exec <AGENT_POD> -- cat /etc/datadog-agent/conf.d/redisdb.d/conf.yaml
```

Here you can see that all it has is what we defined in the `data` section of the ConfigMap.

## Values.yaml

ConfigMap based configurations require you to create a separate file, deploy it and mount it into the agent, however you can use the same configurations you define in the `data` section of the ConfigMap and insert it directly into the `values.yaml` file.

To do this go to the `datadog.confd` section in the `values.yaml` file and you essentially put the contents of the `data` section there, example below:

``` yaml
datadog:
  ...
  confd:
    redisdb.yaml: |-
      ad_identifiers:
        - redis
      init_config:
      instances:
        - host: "%%host%%"
          port: "6379"
```

After redeploying the agent you should be able to see that the `redisdb` check is still working.

To clean up the project you can run:

``` bash
# Deleting the default minikube cluster
minikube delete 

# Deleting a custom minikube cluster
minikube delete -p <CLUSTER_NAME>
```

# Conclusion

So here we went over setting up Kubernetes in you machine using `minikube` and how to use `helm` and `kubectl` to interact with you cluster. You also learned how to configure integrations using various methods.

[< Deploying the Agent](./deploy-agent.md) | [Beginning](../README.md)
