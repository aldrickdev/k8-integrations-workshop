# k8-integrations-workshop

This is a simple workshop for learning about the different way you can go about configuring integrations in a Kubernetes environment.

## Prerequisites

Things needed to follow along:

- Docker
  To check if you have `Docker` installed you can run:

  ``` bash
  docker -v
  ```

  If you get something similar to the output below, then you are good to go

  ``` bash
  # OUTPUT
  Docker version 20.10.17, build 100c701
  ```

  If you don't have it installed, you can follow the [Docker Installation Guide](https://docs.docker.com/desktop/install/mac-install/).

- Minikube
  To check if you have `minikube` installed you can run:

  ``` bash
  minikube version
  ```

  If you get something similar to the output below, then you are good to go

  ``` bash
  # OUTPUT
  minikube version: v1.25.2
  commit: 362d5fdc0a3dbee389b3d3f1034e8023e72bd3a7
  ```

  If you don't have it installed, you can run `brew install minikube`.

- Kubectl
  To check if you have `kubectl` installed you can run:

  ``` bash
  kubectl version --short
  ```

  If you get something similar to the output below, then you are good to go

  ``` bash
  # OUTPUT
  Flag --short has been deprecated, and will be removed in the future. The --short output will become the default.
  Client Version: v1.24.0
  Kustomize Version: v4.5.4
  The connection to the server localhost:8080 was refused - did you specify the right host or port?
  ```

  If you don't have it installed, you can run `brew install kubernetes-cli`.

- Helm
  To check if you have `helm` installed you can run:

  ``` bash
  helm version
  ```

  If you get something similar to the output below, then you are good to go

  ``` bash
  # OUTPUT
  version.BuildInfo{Version:"v3.8.2", GitCommit:"6e3701edea09e5d55a8ca2aae03a68917630e91b", GitTreeState:"clean", GoVersion:"go1.18.1"}
  ```

  If you don't have it installed, you can run `brew install helm`.
