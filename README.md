# Deploy Kepler on a K8s Cluster to Monitor Energy Consumption
This repository illustrates how to install and run Kepler (Kubernetes-based Efficient Power Level Exporter) to monitor energy consumption metrics of workloads on a K8s cluster.

## Background
- Kepler (Kubernetes-based Efficient Power Level Exporter) is a Prometheus exporter that measures energy consumption at the container, pod, VM, and process level by reading hardware sensors and attributing power based on resource utilization [1].
- Kepler uses Intel **RAPL** (Running Average Power Limit) sensors to collect energy data from CPU packages, cores, and memory subsystems, then distributes this energy to workloads based on their CPU time consumption.
- **MSR** (Model Specific Registers) are special registers provided by Intel/AMD that expose low-level power information to the operating system.
- **RAPL** uses **MSRs** to report real-time power consumption and Kepler relies on reading RAPL counters via these MSRs.
- If a hypervisor does not expose them to the guest OS, Kepler cannot obtain real power values.
- Kepler is deployed as a **DaemonSet** in Kubernetes: one Kepler pod runs on every node in the cluster. Each pod acts as an exporter, collecting node-level energy and performance metrics via RAPL/MSRs and exposing them to Prometheus.
- When Kepler pods are deployed on VM nodes rather than bare-metal nodes, most hypervisors do not expose real RAPL power metrics from the host CPU to guest kernels. As a result, Kepler Pod inside a VM cannot access actual energy readings and will crash-loop.
- By enabling synthetic power values, i.e., `dev.fake-cpu-meter.enabled: true` in the Kepler ConfigMap and restart the pods, this allows testing Kepler and collecting metrics in VM environments with estimated power measurements.
- Accordingly, deploying Kepler depends on the environment-specific settings:
  - Development: Use fake/estimate CPU meter when RAPL unavailable. (The approach of this repository).
  - Production: Ensure nodes have Intel RAPL support.
  - Cloud: May need different privilege configurations from the cloud provider.

## Prerequisites
- Helm Chart Installation (Recommended for Kubernetes).
- Kubernetes cluster with kubectl configured.
- Prometheus.
- Grafana.

## Setup and Deployment
### 1. Deploy Kepler 
  1- Clone the repository
  ```bash
  git clone https://github.com/sustainable-computing-io/kepler.git
  cd kepler
   ```

  2- Install Kepler using Helm
  ```bash
  helm install kepler manifests/helm/kepler/ --namespace kepler --create-namespace --set namespace.create=false
  ```
  
  3- Update Kepler ConfigMap to enable synthetic power values.
  - Open ConfigMap file.
```bash
kubectl -n kepler edit configmap kepler
```
    
  - Edit ConfigMap by enabling `fake-cpu-meter`.
```bash
dev:
fake-cpu-meter:
enabled: true
```  
    
  - Restart Kepler Pods.
```bash
kubectl -n kepler delete pod -l app.kubernetes.io/instance=kepler
```
    
  4- Check Deployment Status
    - Check Kepler namespace. Kepler DaemonSet will generate an exporter (pod) on each node of the K8s cluster (5 nodes), in addition to ClusterIP service.
  
  <img width="1617" height="446" alt="Screenshot 2025-09-22 155009" src="https://github.com/user-attachments/assets/4445d8c2-bea2-40da-a9af-2df32549e248" />

### 2. Prometheus Setup
- Configure Prometheus to scrape Kepler metrics

### 3. Grafana Setup
- Use pre-built Grafana dashboards to visualize Kepler energy metrics.



## References
[1] Sustainable Computing, Kepler: Kubernetes-based Efficient Power Level Exporter. [https://sustainable-computing.io](https://sustainable-computing.io)
