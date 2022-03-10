# Hyper-V (Windows 11) images for development based on linux distributions

This project started when I faced a push-back from [VirtualBox image project](https://github.com/arundeep78/virtualbox_dev_images). I did not think I would need it as I started with Virtual Box. I read some articles about Hyper-V and Virtual Box and found there were more [recommendations for VirtualBox](https://praveenmak.medium.com/why-i-switched-from-wsl-to-virtualbox-6c464d51af3). I did not know what awaits me next or lets say that I was expected to an expert in virtualization and VirtualBox configurations.However after facing issues: network, linux, microk8s, high CPU, I thought to give Hyper-V a try. Maybe negativity about Hyper-V comes from the old age view of Unix vs Windows servers. Let's try 

## Motivation

I started with WSL images in order to learn Kubernetes, docker and development that happens using those environment. Mainly my target was
1. Apache Airflow
2. Python development
3. Flask/fastAPI based API development
4. Database setup 
5. Somewhat of Web development simply to expose data that is being processed using above. It could some JS in Flask e.g.

It actually did work 
1. For python development
2. Jupyter notebooks dev environment worked with and without VS code.
3. to have unix based utilities and interface available to connect to remote system e.g. kubectl, helm, Azure etc
4. VS code integration with WSL and remote containers worked nicely.
5. I could learn basics of devcontainers using Docker and VS Code.


However, it started to fail when I tried to follow steps to configure Kubernetes.

1. [Microk8s failed right away](https://github.com/arundeep78/wsl_debian_dev/blob/master/microk8s_readme.md) as WSL does not support SNAP because it depends on systemd architecture.


So, I decided to try a pure virtual machine environment rather than WSL. I tried VirtualBox earlier, but failed (for me). So, now time to try VirtualBox.

## Why Hyper-V

Very simple, it is free and after VirtualBox I don't know what are my other choices with Windows 11 as my base OS on the laptop . There can be another big debate about which Virtualization toolbox is better and many threads would point out to [VMware](https://www.vmware.com/) or [Parallels](https://www.parallels.com/eu/). But, I just selected [VirtualBox](https://www.virtualbox.org/) for familiarity, price and [this article that I came across while trying to find solutions for MicroK8s problem](https://praveenmak.medium.com/why-i-switched-from-wsl-to-virtualbox-6c464d51af3).


## Activate Hyper-V on Windows 11.

This is a simple and straight forward procedure to follow.

1. Check [system requirements for Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v).
2. Install/configure Hyper-V on Windows. I found this documentation to be quite good.




## Various Hyper-V images

Similar to [WSL project](https://github.com/arundeep78/wsl_debian_dev), I would try to make layers of images to move horizontal or verticle for image selection.



1. [Base VirtualBox image with Debian 11](base_debian_readme.md).