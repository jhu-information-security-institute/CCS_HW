# Homework 3

## K8s ConfigMaps

ConfigMaps are Kubernetes API objects designed to store non-confidential configuration data in key-value pairs. They serve as a mechanism to decouple environment-specific configuration from container images, making applications more portable and flexible. 
Primary uses of ConfigMaps:
- Store environment variables, application settings, and configuration files.
- Separate configuration data from application code.
- Enable configuration updates without rebuilding container images.
- Provide reusable configurations across multiple applications.

### Create and Deploy ConfigMaps

#### From Literal

1. Creating a Kubernetes ConfigMap by providing its key-value data directly as plain text strings in the command line.

    ```bash
    kubectl create configmap name.of-configmap --from-literal=key1Name="Key 1 Value" --from-literal=key2Name="Key 2 Value"
    ```

2. Explore the created `name.of-configmap` ConfigMap using `k9s` or `kubectl`.

#### Using a File

1. Explore the [default.conf](./01_configmaps/default.conf), [index.html](./01_configmaps/index.html), and [nginx.yaml](./01_configmaps/nginx.yaml) files.

2. Create ConfigMaps
    ```bash
    kubectl create configmap nginx-config --from-file=default.conf=./01_configmaps/default.conf
    kubectl create configmap nginx-index --from-file=default.conf=./01_configmaps/index.html
    ```

3. Explore the `nginx-configuration` and `nginx-index` ConfigMaps using `k9s` or `kubectl`.

4. Deploy nginx using a custom `default.conf` configuration

    ```bash
    # create nginx deployment and service (nginx will run in port 8080)
    kubectl apply -f ./01_configmaps/nginx.yaml
    # locally expose nginx service
    kubectl port-forward svc/nginx 8080:8080
    ```

5. Open a browser and go to <http://localhost:8080/> or <http://localhost:8080/index.html>.

6. Navigate inside the nginx pod and look for the `default.conf` and `index.html` files.


> **Note:** ConfigMaps are stored in plaintext and should never contain sensitive information.

[You can learn more about ConfigMaps here](https://kubernetes.io/docs/concepts/configuration/configmap/) and [some tutorials are available here](https://kubernetes.io/docs/tutorials/configuration/).

## K8s Secrets

Secrets are similar to ConfigMaps but specifically designed for sensitive data such as passwords, API keys, TLS certificates, and private keys. While they follow the same key-value structure as ConfigMaps, Secrets have additional security considerations.
Key characteristics of Secrets:
- Data is base64-encoded (not encrypted by default).
- More restrictive access controls through RBAC.
- Support for encryption at rest when properly configured.
- Can be managed with tighter security policies.

### Deploying Secrets

#### From Literal

1. Creating a Kubernetes Secret by providing its key-value data directly as plain text strings in the command line.
    ```bash
    kubectl create secret generic ccs-secret --from-literal=password=1234567
    ```

2. Explore the `ccs-secret` secret using `k9s` or `kubectl`.

#### Using a TLS Certificate Keypair

1. Explore the [default-tls.conf](./02_secrets/default-tls.conf), [index.html](./02_secrets/index.html), and [nginx-tls.yaml](./02_secrets/nginx-tls.yaml) files.

2. Generate self-signed certificates using mkcert, openssl or any other tool of your choice.
    ```bash
    # e.g., creates a key and certificate combo named localhost
    mkcert localhost
    ```

3. Create a secret.
    ```bash
    kubectl create secret tls nginx-tls-certificates --cert=localhost.pem --key=localhost-key.pem
    ```

4. Create new ConfigMaps from the nginx configuration that includes tls certificates.
    ```bash
    kubectl create configmap nginx-configuration-tls --from-file=default.conf=./02_secrets/default-tls.conf
    kubectl create configmap nginx-index-new --from-file=index.html=./02_secrets/index.html
    ```

5. Deploy nginx with tls certificates.
    ```bash
    # create nginx deployment and service (nginx will run in port 8443)
    kubectl apply -f ./02_secrets/nginx-tls.yaml
    # locally expose nginx service
    kubectl port-forward svc/nginx 8443:8443
    ```

6. Open browser and go to <https://localhost:8443/> or use `curl https://localhost:8443 -v`.

7. Navigate inside the nginx pod and look for the `default.conf` and the tls certificate files.

8. Stop port forwarding (`ctrl+c`) and terminate the application.
    ```bash
    kubectl delete secret ccs-secret
    kubectl delete -f ./02_secrets/nginx-tls.yaml
    kubectl delete configmap nginx-configuration-tls
    kubectl delete configmap nginx-index
    kubectl delete configmap nginx-index-new
    kubectl delete secret nginx-tls-certificates
    ```

[You can learn more about secrets here.](https://kubernetes.io/docs/concepts/configuration/secret/)

## K8s Namespaces

Namespaces provide a mechanism for isolating groups of API resources within a single cluster. They create logical boundaries that allow resource names to be unique within a namespace but not across namespaces.
Namespaces are generally used for
- Resource isolation: Separate workloads, teams, or environments logically.
- Name scoping: Allow duplicate resource names across different namespaces.
- Management efficiency: Apply policies and configurations at the namespace level.

### Deploying Namespaces

1. Analyze the [tenant-1.yaml](./03_namespaces/tenant-1.yaml) and [tenant-2.yaml](./03_namespaces/tenant-2.yaml) files.

2. Deploy the required resources for 2 applications.
    ```bash
    # APP 1:
    kubectl apply -f 03_namespaces/tenant-1.yaml
    # APP 2:
    kubectl apply -f 03_namespaces/tenant-2.yaml
    ```

3. Inspect the resources created for the namespaces `tenant-1` and `tenant-2` using `k9s` or `kubectl`.

4. Start port forwarding for both applications.
    ```bash
    # forwarding tenant-1 in first terminal
    kubectl port-forward svc/nginx 8081:8080 -n tenant-1
    # forwarding tenant-2 in second terminal
    kubectl port-forward svc/nginx 8082:8080 -n tenant-2
    ```

    Open browser and go to <http://localhost:8081> and <http://localhost:8082> to verify the running applications.

5. Get a shell into any running container.

    **kubectl:**
    ```bash
    # exec into nginx tenant-1
    kubectl -n tenant-1 exec -it <pod name> -- sh
    # exec into nginx tenant-2
    kubectl -n tenant-2 exec -it <pod name> -- sh
    ```

    **k9s:**
    - Namespace > `tenant-1` > Pods `>` nginx `>` press `<s>`
    - Namespace > `tenant-2` > Pods `>` nginx `>` press `<s>`

6. Install `curl` on both nginx containers.
    ```bash
    apk add curl
    ```

7. Test connectivity between services in two different namespaces.

    - From `tenant-1` to `tenant-2`:
        ```bash
        curl http://nginx.tenant-2.svc.cluster.local:8080
        ```

    - From `tenant-2` to `tenant-1`:
        ```bash
        curl http://nginx.tenant-1.svc.cluster.local:8080
        ```

    Notice how the service `URLs` have the following structure: `http://<service name>.<namespace>.svc.cluster.local:<port>`

8. Stop port forwarding (`<ctrl+c>`) and terminate the applications:
    ```bash
    kubectl delete -f 03_namespaces/tenant-1.yaml
    kubectl delete -f 03_namespaces/tenant-2.yaml
    ```

[Learn more about K8s namespaces here.](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

## K8s Certificate Authority

A Certificate Authority (CA) is a trusted entity that issues and manages digital certificates within a Public Key Infrastructure (PKI). In Kubernetes, CAs establish the foundation of trust for all cluster communications.

### Kubernetes PKI Architecture

1. Primary Kubernetes CA (kubernetes-ca):

    - Root authority for API server authentication.
    - Signs certificates for controller manager and scheduler communications.
    - Establishes the fundamental trust anchor for the cluster.

2. etcd CA (etcd-ca):
    - Dedicated authority for etcd cluster communications
    - Prevents privilege escalation where compromised cluster certificates could expose etcd data

3. Front-proxy CA (kubernetes-front-proxy-ca):
    - Manages certificates for API aggregation and extension servers
    - Provides secure pathways for custom API implementations

### Requesting and Approving Certificates

1. Generate a Private Key

    Elliptic Curve Digital Signature Algorithm (ECDSA) with the P-256 curve.

    ```bash
    openssl ecparam -name prime256v1 -genkey -noout -out private.key
    ```

    Optionally you could use a different algorithm, e.g., RSA.

    ```bash
    openssl genrsa -out private.key 2048
    ```

    A file named `private.key` will be created in the active/specified directory.

2. Explore the [cert.cnf](./04_k8s_cert_authority/cert.cnf) file to find the configurations used to generate a Certificate Signing Request.

3. Generate a Certificate Signing Request (CSR).

   ```bash
   openssl req -new -config ./04_k8s_cert_authority/cert.cnf -key private.key -out kubernetes-security.csr
   ```

   > **Note:** Use <https://www.sslshopper.com/csr-decoder.html> or `openssl req -in kubernetes-security.csr -noout -text` to verify the generated `csr`.

4. Encode the generated CSR in Base64 and copy it to your clipboard.

   ```bash
   cat kubernetes-security.csr | base64 | tr -d "\n"
   ```

   > Base64 encoding ensures format compatibility and data integrity for Kubernetes CSR submission.

5. Open the [csr.yaml](./04_k8s_cert_authority/csr.yaml) file, read through it, and paste the encoded certificate in the placeholder at `spec.request` field.

6. Create the `CSR` resource in Kubernetes.

   ```bash
   kubectl apply -f ./04_k8s_cert_authority/csr.yaml
   ```

7. Using `k9s` or `kubectl` list and inspect the created `CSR` resource.

   ```bash
   kubectl get csr -A
   ```

8. Manually approve the CSR.

   ```bash
   kubectl certificate approve kubernetes-security-csr
   ```

9. Retrieve the Signed Certificate.

   ```bash
   kubectl get csr kubernetes-security-csr -o jsonpath='{.status.certificate}'| base64 -d > public.crt
   ```

   When the `CSR` is approved, the new certificate will be issued by the Kubernetes CA and would be found under the `status.certificate` field of the `kubernetes-security-csr`.

10. Verify the `public.crt` certificate was issued by `kubernetes` using <https://www.sslchecker.com/certdecoder> or the `openssl` command.

   ```bash
   cat public.crt | openssl x509 -noout -text
   ```

Now you can use this keypair to configure TLS for your workloads, similar to what you did for [Secrets](#k8s-secrets).

You can learn more here:
 - [PKI certificates and requirements](https://kubernetes.io/docs/setup/best-practices/certificates/).
 - [Certificates and Certificate Signing Requests](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/).

## cert-manager
`cert-manager` is a cloud-native certificate management tool that automatically issues and renews X.509 certificates for your Kubernetes or [OpenStack](https://www.openstack.org/) resources. It operates as a Kubernetes operator, extending the API with certificate-related custom resources.

Automated Operations:
- Certificate issuance: Automatically obtains certificates from configured issuers.
- Lifecycle management: Ensures certificates remain valid and renews them before expiration.
- Secret integration: Stores issued certificates as Kubernetes Secrets.

### Using `cert-manager`

1. Explore the [cert-manager.yaml](./05_cert_manager/cert-manager.yaml), [custom-ca-secret.yaml](./05_cert_manager/custom-ca-secret.yaml), [custom-ca.yaml](./05_cert_manager/custom-ca.yaml), and [default-tls-certificate.yaml](./05_cert_manager/default-tls-certificate.yaml) files.

2. Install `cert-manager` using the default yaml configuration.

   ```bash
   kubectl apply -f 05_cert_manager/cert-manager.yaml
   ```

3. Provide or generate your own `root` Certificate Authority (CA), i.e.:

    ```bash
    # generate private key
    openssl genrsa -out rootCAKey.pem 2048
    # generate public key
    openssl req -x509 -sha256 -new -nodes -key rootCAKey.pem -days 3650 -out rootCACert.pem
    ```

4. `base64` encode the public and private keys for your root CA.

   ```bash
   cat rootCACert.pem | base64 -w 0 # copy to clipboard
   cat rootCAKey.pem | base64 -w 0 # copy to clipboard
   ```

5. Open the [custom-ca-secret.yaml](./05_cert_manager/custom-ca-secret.yaml) file and place the base64 encoded values for the public and private key in the `tls.crt` and `tls.key` fields, then create create the custom ca on kubernetes.

    ```bash
    # create secret
    kubectl apply -f ./05_cert_manager/custom-ca-secret.yaml
    # create cert-manager ClusterIssuer
    kubectl apply -f ./05_cert_manager/custom-ca.yaml
    ```

6. Verify the ca was added to the cluster issuers list.

   ```bash
   kubectl get clusterissuers
   ```

7. Generate a new `tls` certificate using `cert-manager` and your root CA via `cert-manager`.

   ```bash
   kubectl apply -f ./05_cert_manager/default-tls-certificate.yaml
   ```

8. Verify the certificate was generated correctly.

   ```bash
   # check status of the certificate request
   kubectl get certificaterequests
   # check status of the certificate itself
   kubectl get certificates
   # check the tls certificate stored directly on the k8s secret
   kubectl describe secrets default-tls-certificate-secret
   # inspect the public and private keys of the generated certificate
   kubectl get secrets default-tls-certificate-secret -o yaml
   ```

   You can use <https://www.sslshopper.com/certificate-decoder.html> or <https://www.sslchecker.com/certdecoder> to verify the content of the certificate

9. Use this keypair to configure TLS for your workloads, similar to what you did for [Secrets](#k8s-secrets).

10. Terminate the deployed resources.

    ```bash
    kubectl delete secret default-tls-certificate-secret
    kubectl delete -f ./05_cert_manager/default-tls-certificate.yaml
    kubectl delete -f ./05_cert_manager/custom-ca-secret.yaml
    kubectl delete -f ./05_cert_manager/custom-ca.yaml
    kubectl delete -f ./05_cert_manager/cert-manager.yaml
    ```

[You can learn more about cert-manager here.](https://cert-manager.io/docs/)

## Security Context

Security Context is a feature that enables configuration of permission and security settings for pods and containers. It controls security-sensitive aspects of the container runtime environment, implementing the principle of least privilege.

**Key Security Context Settings**
1. User and privilege management:

    - `runAsUser` and `runAsGroup`: Define user and group IDs to prevent running as root
    - `runAsNonRoot`: Ensures containers don't run with root privileges
    - `allowPrivilegeEscalation`: Controls whether processes can gain additional privileges

2. Filesystem and capability controls:

    - `readOnlyRootFilesystem`: Makes the container's root filesystem read-only
    - `capabilities`: Add or drop specific Linux capabilities
    - `privileged`: Controls whether containers run in privileged mode

3. Advanced security features:

    - `seccomp`: Restricts system calls available to containers
    - `AppArmor`: Provides additional access control through mandatory access control policies

### Deploying Pod-Level Security Context

1. Inspect and compare the files [ubuntu.yaml](./06_pod_security_context/ubuntu.yaml) and [ubuntu-with-security-context.yaml](./06_pod_security_context/ubuntu-with-security-context.yaml).

2. Deploy ubuntu pod.

    ```bash
    # create ubuntu pod
    kubectl apply -f ./06_pod_security_context/ubuntu.yaml
    ```

3. Exec into the running container using either `kubectl` or `k9s`.

4. Run the `whoami` command and try to install some applications. E.g., 

    ```bash
    apt-get update && apt-get install curl -y
    ```

5. Terminate the pod.

    ```bash
    kubectl delete -f ./06_pod_security_context/ubuntu.yaml
    ```

6. Deploy pod with `security context` and exec into it.

    ```bash
    kubectl apply -f ./06_pod_security_context/ubuntu-with-security-context.yaml
    ```

7. Run the `whoami` command and try to update system packages.

8. Terminate the pod.

    ```bash
    kubectl delete -f ./06_pod_security_context/ubuntu-with-security-context.yaml
    ```

You can learn more here:
- [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/).
- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/).

## Terrascan

Terrascan is a static code analysis tool designed for scanning Infrastructure as Code (IaC) templates and configurations. It identifies security vulnerabilities, compliance violations, and best practice issues before infrastructure provisioning.

```bash
# Initialize `terrascan`
terrascan init
# Explore `terrascan` commands
terrascan
```

Use `terrascan` to scan for vulnerabilities in the [minio application](./07_terrascan/).
```bash
terrascan scan -i k8s ./07_terrascan/
```

[Learn more about terrascan here](https://github.com/tenable/terrascan). Also check out [07_terrascan/README.md](./07_terrascan/README.md).