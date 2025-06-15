# Local Kubernetes Development Environment with Kind and MetalLB

This project provides a set of configurations and an Ansible playbook to quickly set up a local Kubernetes cluster using [Kind (Kubernetes in Docker)](https://kind.sigs.k8s.io/). It includes the installation and configuration of [MetalLB](https://metallb.universe.tf/) to provide LoadBalancer services, and deploys a sample `nginx` application with persistent storage.

This setup is ideal for local development and testing of Kubernetes applications that require external access and persistent data.

## Prerequisites

Before you begin, ensure you have [Ansible](https://www.ansible.com/) installed on your local machine. The playbook will handle the installation of `kind` and `kubectl` if they are not found.

## File Structure

```
.
├── data
│   ├── worker-1
│   │   └── index.html
│   ├── worker-2
│   └── worker-3
├── kind-cluster.yml
├── kind-config.yaml
├── metallb-config.yaml
├── pv.yaml
├── storageclass.yaml
└── test-app.yaml
```

## Configuration Files

- **`kind-cluster.yml`**: The main Ansible playbook that automates the creation, recreation, and deletion of the Kubernetes cluster and its components.
- **`kind-config.yaml`**: Defines the `kind` cluster topology, which consists of one control-plane node and two worker nodes. It also configures `extraMounts` to mount local `./data/worker-*` directories into the worker nodes.
- **`metallb-config.yaml`**: Configures MetalLB with an `IPAddressPool` to assign external IP addresses to services of type `LoadBalancer`.
- **`storageclass.yaml`**: Defines a `StorageClass` named `local-storage` that enables the use of local persistent volumes.
- **`pv.yaml`**: Creates a `PersistentVolume` that uses the local path `/data` on one of the worker nodes (`dev-cluster-worker`).
- **`test-app.yaml`**: Contains the Kubernetes manifests for a sample `nginx` application, including a `PersistentVolumeClaim` to request storage, a `Deployment` to run the `nginx` container, and a `Service` of type `LoadBalancer` to expose it.

## Usage

The Ansible playbook `kind-cluster.yml` is used to manage the entire lifecycle of the local cluster.

### Create the Cluster

To create the `kind` cluster and deploy all the components, run the following command:

```bash
ansible-playbook kind-cluster.yml --tags create
```

This will:

1. Check for and install `kind` and `kubectl` if they are not present.
2. Create a new `kind` cluster based on `kind-config.yaml`.
3. Install MetalLB.
4. Apply the `StorageClass` and `PersistentVolume` configurations.
5. Deploy the `nginx` test application.

### Recreate the Cluster

To delete the existing cluster and create a new one from scratch, use the `recreate` tag:

```bash
ansible-playbook kind-cluster.yml --tags recreate
```

### Delete the Cluster

To delete the `kind` cluster, run:

```bash
ansible-playbook kind-cluster.yml --tags delete
```

## Accessing the Test Application

Once the cluster is created and the test application is deployed, you can access the `nginx` welcome page. First, get the external IP address assigned to the `nginx-service` by MetalLB:

```bash
kubectl get service nginx-service
```

You will see an output similar to this:

```
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
nginx-service   LoadBalancer   10.96.162.242   172.18.255.200   80:30080/TCP   5m
```

Open your web browser and navigate to the `EXTERNAL-IP` address (in this example, `http://172.18.255.200`). You should see the default `nginx` page.

The `index.html` file served by `nginx` is mounted from the `./data/worker-1/` directory on your local machine. You can modify this file to see live changes.

## License

This project is licensed under the Apache License, Version 2.0.

```
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    [http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
