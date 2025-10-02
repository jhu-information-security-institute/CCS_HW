# Homework 4

## Limiting Pod Resources

Resource limits prevent any single pod from consuming excessive resources and starving other workloads of necessary compute power. These limits work in conjunction with resource requests, which specify the minimum guaranteed resources for a container.

1. Compare the differences between
    - [mem-testing.yaml](./01_pod_resource_limits/mem-testing.yaml) and [mem-testing-limits.yaml](./01_pod_resource_limits/mem-testing-limits.yaml)
    - [cpu-testing.yaml](./01_pod_resource_limits/cpu-testing.yaml) and [cpu-testing-limits.yaml](./01_pod_resource_limits/cpu-testing-limits.yaml)

2. Install metrics server in your cluster.

    ```bash
    kubectl apply -f ./01_pod_resource_limits/metrics-server.yaml
    ```

    After the service is running restart `k9s` and look for the new `CPU` and `Memory` metrics available for your cluster. Look at metrics using `kubectl`:

    ```bash
    kubectl top pod -A
    ```

3. Memory testing

    ```bash
    # deploy container without limits
    kubectl apply -f ./01_pod_resource_limits/mem-testing.yaml
    # look at memory consumtion by the pod
    watch kubectl top pod -l app=mem-testing
    ```

    Update memory stress container to limit the amount of memory to only 100mb.

    ```bash
    # deploy container with limits
    kubectl apply -f ./01_pod_resource_limits/mem-testing-limits.yaml
    ```

    Observe how the container is stopped with status `OOMKilled`.

4. CPU testing

    ```bash
    # deploy container without limits
    kubectl apply -f ./01_pod_resource_limits/cpu-testing.yaml
    # look at cpu consumtion by the pod
    watch kubectl top pod -l app=cpu-testing
    ```

    > The `-cpus "2"` argument in the `yaml` file tells the Container to attempt to use 2 CPUs.

    Update cpu stress container to limit the amount of cpu to only 1 core.

    ```bash
    # deploy container with limits
    kubectl apply -f ./01_pod_resource_limits/cpu-testing-limits.yaml
    ```

    Observe how the container is limited to consume maximum 1 cpu.

5. Terminate all deployed resources.

    ```bash
    kubectl delete -f ./01_pod_resource_limits/metrics-server.yaml
    kubectl delete -f ./01_pod_resource_limits/mem-testing.yaml
    kubectl delete -f ./01_pod_resource_limits/cpu-testing.yaml
    ```

[You can learn more about limiting resources here.](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pod-level-resources/)

## Privilege Escalation Using Docker Containers

1. Create a new file as the `root` user.

    ```bash
    sudo -i # login as root
    echo "supersecret" > /tmp/secret.txt
    chmod 600 /tmp/secret.txt
    ```

2. List files and read `secret.txt` content as root.

    ```bash
    # list files in current directory
    ls -lhr /tmp/secret.txt
    # show content of file
    cat /tmp/secret.txt
    exit # logout from root
    ```

3. Using a regular user (non-root) account try to display the content of the `secret.txt` file.

    ```bash
    cat /tmp/secret.txt
    ```

4. Using a regular user (non-root) account run a docker container and mount the `secret.txt` file to read the content.

    ```bash
    docker run -v "/tmp/secret.txt:/tmp/secret.txt" -it alpine sh -c "cat /tmp/secret.txt"
    # you can do the same with the /etc/shadow file
    docker run -v "/etc/shadow:/tmp/shadow" -it alpine sh -c "cat /tmp/shadow"
    ```

5. Inspect who's running the docker container using the `ps` command.

    ```bash
    # using a regular user account run the following
    docker run -it alpine sh -c "sleep 3600"
    # in a different terminal run the ps command
    docker ps -a
    # replace $CONTAINER_ID for the actual container id
    ps -aux | grep $CONTAINER_ID
    # stop the alpine container
    docker stop $CONTAINER_ID
    ```

## kube-bench: CIS Kubernetes Security Benchmarking Tool

`kube-bench` is an open-source security auditing tool that automatically checks Kubernetes clusters against the [Center for Internet Security (CIS) Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes).

1. Deploy `kube-bench` into your cluster do start the assessment.

    ```bash
    kubectl apply -f 02_kube-bench/kube-bench.yaml
    ```

2. Confirm `kube-bench` pod was created and status is `Completed`.

    ```bash
    kubectl get pods
    ```

3. Inspect `kube-bench` report in pod logs.

    **kubectl:**

    ```bash
    kubectl logs -l app=kube-bench --tail=-1    # Analyze the output.
    ```

    **k9s:**

    - Pods `>` kube-bench `>` press `<l>`

4. Terminate the deployed resources.

    ```bash
    kubectl delete -f 02_kube-bench/kube-bench.yaml
    ```

## KubeLinter

KubeLinter is an open-source static analysis tool designed to check Kubernetes YAML files and Helm charts for security and configuration best practices before deployment. Its main objective is to catch misconfigurations, vulnerabilities, and deviations from production-readiness standards early in the development process.

Helm charts are standardized package formats that describe, configure, and automate the deployment of applications onto Kubernetes clusters.

1. Run `kube-linter` command and get familiar with it.

    ```bash
    kube-linter
    ```

2. Use `kube-linter` to scan for vulnerabilities in the `wordpress` application.

    ```bash
    kube-linter lint 03_KubeLinter/wordpress/*.yaml
    # report in json format
    kube-linter lint 03_KubeLinter/wordpress/*.yaml --format json
    # filter specific fields using jq
    kube-linter lint 03_KubeLinter/wordpress/*.yaml --format json | jq -r '[.Reports[] | { "Check": .Check, "Service": .Object.K8sObject.Name, "Type": .Object.K8sObject.GroupVersionKind.Kind, "Message": .Diagnostic.Message, "Remediation": .Remediation }]'
    ```

3. Use `kube-linter` to scan for vulnerabilities in the wordpress `helm` application.

    ```bash
    kube-linter lint 03_KubeLinter/wordpress-helm-chart/
    ```

## Service Account Tokens

Service account tokens in Kubernetes are cryptographic credentials automatically generated and managed by the cluster, used to provide an identity for workloads to enable secure interaction with the Kubernetes API and other services. These tokens are usually mounted inside Pods and take the form of time-limited JSON Web Tokens, enabling applications running in the cluster to authenticate themselves as a Kubernetes "service account" rather than a human user.

1. Deploy ubuntu pod.

    ```bash
    kubectl apply -f 04_service_account_token/ubuntu.yaml
    ```

2. Exec into the running container.

3. Install `curl` and `jq`.

    ```bash
    apt-get update && apt-get install curl jq -y
    ```

4. Move to the `serviceaccount` folder.

    ```bash
    cd /var/run/secrets/kubernetes.io/serviceaccount
    ```

5. Analyze the 3 files under the `serviceaccount` folder
    - ca.crt
    - namespace
    - token

6. Visualize `token` using <https://jwt.io/> or a similar tool.

7. Query the Kubernetes api server.

    ```bash
    curl https://kubernetes.default.svc.cluster.local
    curl https://kubernetes.default.svc.cluster.local -k
    # pass ca.crt to verify tls connection
    curl https://kubernetes.default.svc.cluster.local --cacert ca.crt
    ```

8. Authenticate using the service account `token`.

    ```bash
    export TOKEN=$(cat token)
    curl --cacert ca.crt https://kubernetes.default.svc.cluster.local/api/v1/namespaces?limit=500 -H "Authorization: Bearer $TOKEN"
    # Use jq to parse the list of existing namespaces in the cluster
    curl --cacert ca.crt https://kubernetes.default.svc.cluster.local/api/v1/namespaces?limit=500 -H "Authorization: Bearer $TOKEN" | jq ".items[].metadata.name"

    exit
    ```

9. Deploy `ubuntu-no-sa` pod.

    ```bash
    # delete ubuntu pod
    kubectl delete -f 04_service_account_token/ubuntu.yaml
    # create ubuntu pod without mounting service account by default
    kubectl apply -f 04_service_account_token/ubuntu-no-sa.yaml
    ```

10. Try to navigate again to the `serviceaccount` folder (You should get an error).

    ```bash
    kubectl exec -it pod/ubuntu -- /bin/bash
    cd /var/run/secrets/kubernetes.io/serviceaccount

    exit
    ```

11. Terminate the deployed pod.

    ```bash
    kubectl delete -f 04_service_account_token/ubuntu-no-sa.yaml
    ```

[Learn more about configuring service accounts here.](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

## Calico: Network Policy and Security for Kubernetes

Calico is a comprehensive networking and network security solution for Kubernetes that provides advanced network policy enforcement, IP address management, and Container Network Interface (CNI) capabilities.

1. Look at [tenant-1.yaml](./05_calico_network_policies/tenant-1.yaml) file and deploy all the resources for application 1.

    ```bash
    kubectl apply -f 05_calico_network_policies/tenant-1.yaml
    ```

2. Look at [tenant-2.yaml](./05_calico_network_policies/tenant-2.yaml) file and deploy all the resources for application 2.

    ```bash
    kubectl apply -f 05_calico_network_policies/tenant-2.yaml
    ```


3. Inspect the resources created for the `tenant-1` and `tenant-2` namespaces using `k9s` or `kubectl`.

    ```bash
    # tenant-1
    kubectl get all --namespace tenant-1
    # tenant-2
    kubectl get all --namespace tenant-2
    ```

4. Exec into the running containers.

5. Install `curl` on both nginx containers.

    ```bash
    apk add curl
    ```

6. Test connectivity between services in two different namespaces.

    ```bash
    # From `tenant-1` to `tenant-2`
    curl http://nginx.tenant-2.svc.cluster.local:8080
    exit

    # From `tenant-2` to `tenant-1`
    curl http://nginx.tenant-1.svc.cluster.local:8080
    exit
    ```

    > Notice how the service URLs have the following structure: `http://<service name>.<namespace>.svc.cluster.local:<port>`.

7. Install the Tigera Calico operator and custom resource definitions.

    ```bash
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
    ```

8. Install Calico by creating the necessary custom resource.

    > Before creating this [manifest](./05_calico_network_policies/custom-resources.yaml), read its contents and make sure its settings are correct for your environment. For example, you may need to change the default IP pool CIDR to match your pod network CIDR. See <https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart>.

    ```bash
    kubectl create -f 05_calico_network_policies/custom-resources.yaml
    ```

9. Confirm that all of the pods are running with the following command.

    ```bash
    watch kubectl get pods -n calico-system
    ```

10. Block all incomming requests to `tenant-2` namespace workloads using Calico Networking Policies. Look at [np-default-deny.yaml](./05_calico_network_policies/np-default-deny.yaml) and then run:

    ```bash
    kubectl apply -f 05_calico_network_policies/np-default-deny.yaml
    ```

    > **Note:** In some cluster setups nginx pods need to be restarted.
    > ```bash
    > kubectl rollout restart deployment nginx -n tenant-2
    > ```
    > You will have to `apk add curl` again after getting a shell into the restarted pod.

11. Exec into the `tenant-1` running container again and test connectivity to the running container in `tenant-2`. You should see a timeout error.

    ```bash
    # -m option adds a timeout to curl. 5 secs added here.
    curl -m 5 http://nginx.tenant-2.svc.cluster.local:8080
    exit
    ```

12. Exec into the `tenant-2` running container again and test connectivity to the running container in `tenant-1`.

    ```bash
    # request to tenant-1
    curl -m 5 http://nginx.tenant-1.svc.cluster.local:8080
    # request to itself using service name should timeout
    curl -m 5 http://nginx.tenant-2.svc.cluster.local:8080
    # request to itself using localhost
    curl -m 5 http://localhost:8080

    exit
    ```

13. Deploy networking rule to allow internal connectivity for `tenant-2` namespace.

    ```bash
    kubectl apply -f 05_calico_network_policies/np-allow-namespace-connectivity.yaml  
    ```

    Exec into the `tenant-2` running container again and test connectivity

    ```bash
    # request to itself using service name should work this time
    curl -m 5 http://nginx.tenant-2.svc.cluster.local:8080
    ```

    Exec into the `tenant-1` running container again and test connectivity to `tenant-2`. It should still return an error.

14. Deploy a new tenant namespace. Update existing networking rule to allow connectivity from workloads running on `tenant-3` namespace to `tenant-2` namespace.

    ```bash
    # deploy tenant-3
    kubectl apply -f 05_calico_network_policies/tenant-3.yaml
    # update tenant-2 networking rule to accept tenant-3 connections
    kubectl apply -f 05_calico_network_policies/np-allow-namespace-connectivity-update.yaml  
    ```

    Exec into the `tenant-3` running container and install `curl`.
    
    ```bash
    # request to tenant-2 should work this time
    curl -m 5 http://nginx.tenant-2.svc.cluster.local:8080
    ```

    Exec into the `tenant-1` running container again and test connectivity to `tenant-2`. It should still return an error.

15. Terminate all deployed resources.

    ```bash
    kubectl delete -f 05_calico_network_policies/np-default-deny.yaml
    kubectl delete -f 05_calico_network_policies/np-allow-namespace-connectivity.yaml
    kubectl delete -f 05_calico_network_policies/np-allow-namespace-connectivity-update.yaml
    kubectl delete -f 05_calico_network_policies/tenant-1.yaml
    kubectl delete -f 05_calico_network_policies/tenant-2.yaml
    kubectl delete -f 05_calico_network_policies/tenant-3.yaml
    kubectl delete -f 05_calico_network_policies/custom-resources.yaml
    kubectl delete -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
    ```

You can learn more here:
- [Kubernetes Policy](https://docs.tigera.io/calico/latest/network-policy/get-started/kubernetes-policy/)
- [Tigera - Network policy](https://docs.tigera.io/calico/latest/network-policy/)