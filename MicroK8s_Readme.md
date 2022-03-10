# Hyper-V Debian image with Microk8s installation

This documentation is to have configure a Debian 11 image in Hyper-V with microk8s installed on the image. This is based on the [standard Debian 11 image configured with basic tools like Oh-my-ZSH etc](base_debian_readme.md).

I decided to create a separate image simply not to have any conflict with other flavours of K8s. Microk8s uses SNAP package and I read somewhere that one of the flavours(I think it was Kind) does not like snap. Maybe those issues are solved, but maybe it is better to keep them isolated lets continue to have a isolation. Afterall all this containerization is about isolation, right?


## Why MicroK8s?

The documentation says that it has a very low memory footprint. This is cool as it should help keep some resources free on my laptop. 

It was also the image mentioned in one of the [Microsoft Tutorails](https://docs.microsoft.com/en-us/learn/modules/intro-to-kubernetes/) that I followed to learn K8s. 

Additionally, it does not seems to need docker to run it. At the moment, I have [issues with Kind and Minikube clusters in WSL](https://github.com/arundeep78/wsl_debian_dev/blob/master/Kind_k8s_Readme.md#check-internet-access-from-nodes) as clusters nodes cannot reach docker repository, even though all other internet sites are accessible! So, it might help me to get going with actual cluster tasks rather than figuring out why I cannot reach docker repository.

After WSL I [tried Virtual Box](https://github.com/arundeep78/virtualbox_dev_images/blob/master/MicroK8s_Readme.md), but it failed as well. Somehow VM would stop working even though microk8s snap would finish with success!

## Create a new Hyper-V VM machine

One can prepare a new VM from scratch. But in this case, I will simply clone the base image using Hyper-V tools. This image has all the basic tools needed for MicroK8s.

Configure the VM to have
- 4GB RAM
- atleast 2vcpu

## Install and configure MicroK8s

### install mikrok8s

Follow [official documentation](https://microk8s.io/?ref=thechiefio) for basic installation steps.

```zsh
$ sudo snap install microk8s --classic

microk8s (1.23/stable) v1.23.3 from Canonical✓ installed
```

Even though SNAP ends with a success. But the microk8s services takes some time to get it all working. I do not know what all it does, but I noticed e.g. `~/.kube` folder took about 5-10min to get created.

### make sure snap binaries path is set

This is an issue with SNAP and ZSH. if you are using bash, then it will work fine. 

Steps are given in the [base image steps](base_debian_readme.md)

### Configure non-root user to execute microk8s commands

```zsh
sudo usermod -aG microk8s $USER
sudo chown -f -R $USER ~/.kube

#important to make sure the user has access
newgrp microk8s
```

### enable helm as an add-on

```zsh
microk8s enable helm
```

### Add microk8s plugin to OhMyZSH

update `~/.zshrc`

```zsh

plugins = (
  #other plugins
  microk8s
)
```

### Check microK8s status

```zsh
$ microk8s status --wait-ready
microk8s is running
high-availability: no
  datastore master nodes: 127.0.0.1:19001
  datastore standby nodes: none
addons:
  enabled:
    ha-cluster           # Configure high availability on the current node
    helm                 # Helm 2 - the package manager for Kubernetes
  disabled:
    ambassador           # Ambassador API Gateway and Ingress
    cilium               # SDN, fast with full network policy
    dashboard            # The Kubernetes dashboard
    dashboard-ingress    # Ingress definition for Kubernetes dashboard
    dns                  # CoreDNS
    fluentd              # Elasticsearch-Fluentd-Kibana logging and monitoring
    gpu                  # Automatic enablement of Nvidia CUDA
    helm3                # Helm 3 - Kubernetes package manager
    host-access          # Allow Pods connecting to Host services smoothly
```

## Enable and access kubernetes dashboard

There is a good [documentation available here](https://adamtheautomator.com/microk8s/) for reference.

### Enable dashboard add-on

```zsh
microk8s enable dashboard
```

### Get IP of the dashboard service

```zsh
$microk8s kubectl get service --namespace kube-system
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
metrics-server              ClusterIP   10.152.183.97    <none>        443/TCP    2m17s
kubernetes-dashboard        ClusterIP   10.152.183.161   <none>        443/TCP    2m9s
dashboard-metrics-scraper   ClusterIP   10.152.183.184   <none>        8000/TCP   2m9s
```

### Get token to access the dashboard page

```zsh
$ token=$(microk8s kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
microk8s kubectl -n kube-system describe secret $token

Name:         default-token-9k922
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 65d129e5-1535-4299-bea4-df5825f4256a

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1123 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImpQSTdaOXBtVVA2aHpKZVdMcHRWcFdiejR3QWR4NW5uZU84Zk5tOFpZb2MifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkZWZhdWx0LXRva2VuLTlrOTIyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImRlZmF1bHQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI2NWQxMjllNS0xNTM1LTQyOTktYmVhNC1kZjU4MjVmNDI1NmEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06ZGVmYXVsdCJ9.lM_NWQMGtJRjR36imAe2KrgJaCzemZ5eWjQU7yhF8QKSPQdJ8okerLf7mFWkzpi5ixA7tVhUA2sKefEeVfmX3DxiNlNns36zav68CAGW483P5Svwnw5vLkwaZ_qxaVewYC_NxvKsn6w5uoLVLMjzIDPKAosExzrHdvOiG1XhEPGp-wvtaFUY55hN45Vr0CTgDb-1HO8a7Ed8RNH9mNTQrr1_t1lu6_-AbmdmwCkGCNMgWtheUHwK9s4LUlP0GLQaE9Zoah5syoEYbVNneidaQP4ZWoOJtBx1WCSmAAgfzeS19IGglUH5_Uon6f4JeTeSepy0--OL3VjpqZXI2fihLA
```

### Port forwad of the dashboard service.

This was a bit tricky. All the basic tutorials seems to assume that microk8s is being installed directed on the OS or VM with a graphic interface. In my case, I have Debian VM running on a windows host. microk8s is configured on Debian, but I need to access kubernetes services/apps e.g. dashboard from windows browser. By default, it does not work.

e.g the [IP address obtained above](#get-ip-of-the-dashboard-service) cannot be directly used in windows host. as that IP is not reachable.

**NOTE**: This can also be due to type of network configured for Hyper-VM. But that's something I do not know. 

So, to access the service, I had to create a port-forward to the dashboard service with binding to the general IP address. 

```zsh
microk8s.kubectl port-forward --address 0.0.0.0 svc/kubernetes-dashboard --namespace kube-system 8443:443
```

**NOTE**: Hyper-V VM has multiple IP addresses now due to microk8s. However, only 1 is accessible from Windows host. 

I can now access dashboard using IP address of Hyper-V VM accessible from windows. In this case it was 172.27.18.105.

Also, if I had to use HTTPS protocaol and bypass the certificate error. Direct HTTP access was not possible.

**Direct HTTP access**: no response

 ![HTTP access to dashboard error](images/microk8s/http_access_dashboard_error.drawio.svg)

 **HTTPS access**: need to bypass certification warning

 ![HTTPs access to the URL gives certificate warning](images/microk8s/https_access_certificate_error.drawio.svg)

 But, if I continue, it gives me finally the dashboard page.

 ![k8s dashboard page](images/microk8s/k8s_dashboard.drawio.svg)


 paste the token value that was obtained above and sign in to the dashboard.

 finally the satisfaction to the dashboard page working.

 ![Kubernetes dashboard page is accessible on windows host](images/microk8s/k8s_dashboard_accessible.drawio.svg)



## Memory usage

- Memory usage within the VM is not too high.

![Memory usage inside VM](images/microk8s/linux_mem_usage.drawio.svg)

- Hyper-V VM memory usage
I was surprised to see that my laptop was running at 95% memory usage in Windows. And I have 32GB on my laptop!! They way Hyper-V works, standard task manager in Windows cannot track Hyper-V memory usage.

I found that all of the memory was used by Hyper-V VM. Checking VM settings revealed that dynamic memory was setup for VM with upper limit as 100GB. 
1. I first tried to limit it to 16GB. In 5-10 min without doing anything VM captured 16GB of the memory.
2. Then I decided to stop dynamic allocation and fix it to 4GB based on the current usage it seems good enough.


Maybe there is some bug in the Windows 11 Hyper-V or some settings that I need to look for. But, for now this is working and I am happy to close this file with a smile.


In comparison to WSL images Hyper-V VM .vhdx files are really huge. This one is 6GB already!

## Optional: Install kubectl 

microk8s has an inbuilt kubectl, but it is not so good with command completion. SO, I decided to install kubectl directly.

There are [different ways to install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/). Below is the procedure using binaries

```zsh
# Download Kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Install Kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

- configure OhMyZSH plugin. update `~/.zshrc` file

```zsh
plugins = (
  #other plugins
  kubectl
)
```

- Generate kube config file for microk8s

```zsh
cd $HOME
mkdir .kube # if the directory already does not exist
cd .kube
microk8s config > config
```

This makes getting pods as easy as 

```zsh
$kgp -A
NAMESPACE     NAME                                         READY   STATUS    RESTARTS      AGE
kube-system   dashboard-metrics-scraper-69d9497b54-mzp6z   1/1     Running   2 (42m ago)   149m
kube-system   calico-node-2bfbj                            1/1     Running   3 (42m ago)   11h
kube-system   kubernetes-dashboard-585bdb5648-nd7cg        1/1     Running   4 (37m ago)   149m
kube-system   calico-kube-controllers-68745cbffd-pk2f2     1/1     Running   3 (42m ago)   11h
kube-system   metrics-server-679c5f986d-qk5z7              1/1     Running   3 (37m ago)   150m
```