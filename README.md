# Cloud Computing Security Fall 2025 DVKA Homework Series

## Requirements

### Hardware

- Processors: 1, Cores: 4
- Memory: 4GB
- Disk Space: Approximately 200GB for all content

### Software
- OS: Kali/Ubuntu
- [jq](https://jqlang.github.io/jq/)
- [Docker](https://docs.docker.com/engine/install/)
- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
- [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
- [Kustomize](https://kustomize.io/)
- [k9s](https://k9scli.io/topics/install/)
- [mkcert](https://github.com/FiloSottile/mkcert)
- [kube-bench](https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml)
- [kube-hunter](https://github.com/aquasecurity/kube-hunter)
- [kube-linter](https://github.com/stackrox/kube-linter/releases/download/v0.6.5/kube-linter-linux.tar.gz)
- [terrascan](https://github.com/tenable/terrascan/releases/download/v1.18.3/terrascan_1.18.3_Linux_x86_64.tar.gz)
- [kubeaudit](https://github.com/Shopify/kubeaudit/releases/download/v0.22.0/kubeaudit_0.22.0_linux_amd64.tar.gz)
- [nuclei](https://github.com/projectdiscovery/nuclei/releases/download/v3.2.9/nuclei_3.2.9_linux_amd64.zip)

> **Recommendation**: Create a virtual machine with an OS of your choice (Kali/Ubuntu), allocate the above mentioned hardware resources, and then follow the steps below.

### Install Script

```bash
# make it executable
chmod +x install-tools.sh
# install all required tools
sudo ./install-tools.sh --install
```

Example output:

```bash
Package lists updated .... ✅ 
Prerequisites installed .... ✅ 
Docker installed .... ✅ 
Kind installed .... ✅ 
kubectl installed .... ✅ 
Kustomize installed .... ✅ 
k9s installed .... ✅ 
mkcert installed .... ✅ 
kube-hunter installed .... ✅ 
kube-linter installed .... ✅ 
terrascan installed .... ✅ 
kubeaudit installed .... ✅ 
nuclei installed .... ✅ 
apt autoremove completed .... ✅
```

### Container Images

The [images.txt](./images.txt) file has a list of all the container images that will be required throughout the course.

```bash
# pull all images
for image in $(cat images.txt); do docker pull $image; done;
```

Pulling all images at once could be a time- and space- consuming process. To save time and storage, you may choose to pull them as required. In the following command, replace `<image>` with the actual name of the required image from the [list](./images.txt).

```bash
docker pull <image>
```

#### Verify Images

Verify all the images are in your local registry.

```bash
docker images
```

---
## All DVKA Homeworks

#### [Homework 2 - DVKA Set 1](./Assignments/homework_02/README.md)
#### [Homework 3 - DVKA Set 2](./Assignments/homework_03/README.md)
#### [Homework 4 - DVKA Set 3](./Assignments/homework_04/README.md)

- **Note: Updated minimum [hardware requirements](#hardware). Please make the necessary changes in the settings of your own VM to ensure smoother performance.**

---

## References

1. https://github.com/Alevsk/dvka/
