# Deploy Kepler on a K8s Cluster to Monitor Energy Consumption
This repository illustrates how to install and run Kepler (Kubernetes-based Efficient Power Level Exporter) on a K8s cluster for monitoring energy consumption metrics of workloads.

## Prerequisites
- Helm Chart Installation (Recommended for Kubernetes).
- Kubernetes cluster with kubectl configured.

## Setup and Deployment
  1- Clone the repository
  ```bash
  git clone https://github.com/sustainable-computing-io/kepler.git
  cd kepler
   ```

2- Install Kepler using Helm
  ```bash
  helm install kepler manifests/helm/kepler/ --namespace kepler --create-namespace --set namespace.create=false
  ```
