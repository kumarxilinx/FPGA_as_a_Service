# Xilinx FPGA Plugin Deployment Full Tutorial

This documentation describes how to deploy FGPA plugin with Docker and Kubernetes on CentOS and Ubuntu.
## 1. Install Docker

### Prerequisites

CentOS:

-   A maintained/supported version of CentOS
-   A user account with sudo privileges
-   Terminal access
-   CentOS Extras repository – this is enabled by default, but if yours has been disabled you’ll need to re-enable it
-   Software package installer yum

Ubuntu:

-   Ubuntu operating system
-   A user account with sudo privileges
-   Command-line/terminal
-   Docker software repositories (optional)

### Installing Docker on CentOS 7 With Yum

#### Step 1: Update Docker Package Database

`#sudo yum check-update`

#### Step 2: Install the Dependencies

`#sudo yum install -y yum-utils device-mapper-persistent-data lvm2`

#### Step 3: Add the Docker Repository to CentOS

To install the  **edge**  or  **test**  versions of Docker, you need to add the Docker CE stable repository to your system. To do so, run the command:

```output
#sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

A **stable** release is tested more thoroughly and has a slower update cycle. On the other hand, **Edge** release updates are more frequent but aren’t subject to as many stability tests.

**Note:** If you’re only going to use the stable release, don’t enable these extra repositories. The Docker installation process defaults to the latest version of Docker unless you specify otherwise. Leaving the stable repository enabled makes sure that you aren’t accidentally updating from a stable release to an edge release.

#### Step 4: Install Docker On CentOS Using Yum

With everything set, you can finally move on to installing Docker on CentOS 7 by running:

`#sudo yum install docker`

The system should begin the installation. Once it finishes, it will notify you the installation is complete and which version of Docker is now running on your system.

Your operating system may ask you to accept the GPG key. This is like a digital fingerprint, so you know whether to trust the installation.

#### Step 5: Manage Docker Service

Although you have installed Docker on CentOS, the service is still not running.

To start the service, enable it to run at startup. Run the following commands in the order listed below.

Start Docker:

`#sudo systemctl start docker`

Enable Docker:

`#sudo systemctl enable docker`

Check the status of the service:

`#sudo docker run hello-world`

### Installing Docker on Ubuntu With Apt-get

#### Step 1: Update Software Repositories

`#sudo apt-get update`

#### Step 2: Install Docker

`#sudo apt-get install docker.io`

#### Step 3: Manage Docker Service

Although you have installed Docker on Ubuntu, the service is still not running.

To start the service, enable it to run at startup. Run the following commands in the order listed below.

Start Docker:
`#sudo systemctl start docker`

Enable Docker:

`#sudo systemctl enable docker`

Check the status of the service:

`#sudo docker run hello-world`

## 2. Install Kubernetes

You will install these packages on all of your machines:

-   `kubeadm`: the command to bootstrap the cluster.

-   `kubelet`: the component that runs on all of the machines in your cluster and does things like starting pods and containers.

-   `kubectl`: the command line util to talk to your cluster.


Here is the referred document from Kubernetes:

[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

### Installing kubeadm, kubelet and kubectl on CentOS

#### Step 1: Set kubernetes repo

`#update-alternatives --set iptables /usr/sbin/iptables-legacy`

`#cat /etc/yum.repos.d/kubernetes.repo`  

```
[kubernetes]  
name=Kubernetes  
baseurl=[https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64](https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64)  
enabled=1  
gpgcheck=1  
repo_gpgcheck=1  
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
```

#### Step 2: Set SELinux in permissive mode (effectively disabling it)

`#sudo setenforce 0  `

`#sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config `

**Note**: Setting SELinux in permissive mode by running setenforce 0 and sed ... effectively disables it. This is required to allow containers to access the host filesystem, which is needed by pod networks for example. You have to do this until SELinux support is improved in the kubelet.  

Some users on RHEL/CentOS 7 have reported issues with traffic being routed incorrectly due to iptables being bypassed. You should ensure net.bridge.bridge-nf-call-iptables is set to 1 in your sysctl config, e.g.

`#cat /etc/sysctl.d/k8s.conf  `

net.bridge.bridge-nf-call-iptables = 1  

`#sudo sysctl --system  `

Make sure that the br_netfilter module is loaded before this step. This can be done by running

`#lsmod | grep br_netfilter`

To load it explicitly call

`#sudo modprobe br_netfilter`

#### Step 3: Install Kubernetes

`#sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes`

`#sudo systemctl enable --now kubelet`

### Installing kubeadm, kubelet and kubectl on Ubuntu

#### Step 1: Set kubernetes repo

`#sudo apt-get update`

`#sudo apt-get install -y iptables arptables ebtable`

#### Step 2: Install Kubernetes

```bash
#sudo apt-get update && sudo apt-get install -y apt-transport-https curl
#curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
#sudo apt-get update
#sudo apt-get install -y kubelet kubeadm kubectl
#sudo apt-mark hold kubelet kubeadm kubectl
```
## 3. Configure Cluster

Here will just create **master** node and use it.

### Disable swap

`#sudo swapoff -a`

**Note**: If there is no enough space on system, please try disable swap and remove the swap file.

This command only temporary disable swap, run this command each time after reboot the machine.

### Create and Configure node

#### Step 1: init master node
```
#sudo kubeadm init --pod-network-cidr=10.244.0.0/16
#sudo mkdir -p $HOME/.kube
#sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
#sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**Note:** For issues like: "The connection to the server localhost:8080 was refused - did you specify the right host or port?"

Check port status:

```fallback
#netstat -nltp | grep apiserver
```
Adding environment variable in ~/.bash_porfile

```fallback
#export KUBECONFIG=/etc/kubernetes/admin.conf
```

```fallback
#source ~/.bash_profile
```

#### Step 2: configure flannel


install flannel (for Kubernetes version 1.7+)  

`#sysctl net.bridge.bridge-nf-call-iptables=1  `

`#kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`

For other version, please refer [https://github.com/coreos/flannel](https://github.com/coreos/flannel) to do the configuration.

#### Step 3: check the pod

`#sudo kubectl get pod -n kube-system -o wide`

**NOTE**: If there is multiple AWS instances, then go ahead to create slave node on other instance and use "kubeadm join" to add the slave node into cluster.

## 4. Install Xilinx Runtime

Download XRT from github, build and install it.

### Setup tool

`#scl enable devtoolset-6 bash`

If scl and devtoolset is not installed, then need to install the listed tools.

### Setup AWS FGPA

Here need to download aws FGPA because XRT build will depend on the it.

`#git clone http://github.com/aws/aws-fpga.git`

`#export AWS_FPGA_REPO_DIR="path of aws-fpga"`

### Build and Install XRT

**Note:** Based on current test, XRT 2019.2.0.3 works well on AWS F1, the master version has issue on some F1 instance. So here we recommend to use 2019.2.0.3 version.

#### Step 1: Build XRT
```
#git clone -b 2019.2.0.3 https://github.com/Xilinx/XRT.git
#./src/runtime_src/tools/scripts/xrtdeps.sh
#cd build
#./build.sh
#cd Release
#make package
```
**Note**: Need to make sure $AWS_FPGA_REPO_DIR is set to the right directory of aws-fpga before running build.

#### Step 2: Install XRT

`#yum install xrt_201920.2.3.0_7.7.1908-xrt.rpm`

`#yum install xrt_201920.2.3.0_7.7.1908-aws.rpm`

Please refer to the full instruction on how to build and install XRT:

[https://github.com/Xilinx/XRT/blob/master/src/runtime_src/doc/toc/build.rst](https://github.com/Xilinx/XRT/blob/master/src/runtime_src/doc/toc/build.rst)

`#source /opt/xilinx/xrt/setup.sh`

To check the FPGA device on the system:

```
#systemctl start mpd
#systemctl status mpd
#xbutil scan
```

## 5. Install Kubernetes FPGA Plugin

Here we only have one node (master), and plan to deploy the FPGA plug on this node, to enable this configuration, we need to configure the control plan node.

### Control plane node isolation

By default, your cluster will not schedule Pods on the control-plane node for security reasons. If you want to be able to schedule Pods on the control-plane node, e.g. for a single-machine Kubernetes cluster for development, run:

`#kubectl taint nodes --all node-role.kubernetes.io/master-`

### Install Kubernetes FPGA plugin

####   
Step 1: down plugin source

`#git clone  https://github.com/Xilinx/FPGA_as_a_Service.git`

Deploy FPGA device plugin as daemonset:  

`#kubectl create -f ./FPGA_as_a_Service/k8s-fpga-device-plugin/trunk/fpga-device-plugin.yml `

To check the status of daemonset:  

`#kubectl get pod -n kube-system  `

Get node name:  

`#kubectl get node  `

Check FPGA resource in the worker node:  

`#kubectl describe node nodename  `

You should get the FPGA resources name under the pods information.

#### Step 2: Deploy user pod

`#kubectl create -f mypod.yaml`

**Note:**

1) mypod.yaml is under ./aws
2) modify the image to be "xilinxatg/aws-fpga-verify:20200131"
3) modify the resources: set limits same as the that in worker node like "[xilinx.com/fpga-xilinx_aws-vu9p-f1_dynamic_5_0-43981](http://xilinx.com/fpga-xilinx_aws-vu9p-f1_dynamic_5_0-43981): 1". To run "kubectl describe node nodename" to find out the resource.

To check status of the deployed pod:  

`#kubectl get pod`

#### Step 3: Run the test in pod
after the pod status truns to Running, run hello world in the pod:  

`#kubectl exec -it my-pod /bin/bash  `

`#my-pod>source /opt/xilinx/xrt/setup.sh  `

**Note:**  Need to set the INTERNAL_BUILD=1 if xbutil complain the version not match:  

```
#my-pod>export INTERNAL_BUILD=1  
#my-pod>xbutil scan  
#my-pod>cd /opt/test/  
#my-pod>./helloworld vector_addition_hw.awsxclbin
```
## 6. How to build new docker image

We will use an example to explain how a new docker image with desired contents such as your xclbin, your host code etc. can be built. Please note that any accelerator (FPGA) docker image should be derived form the base docker Xilinx image **xilinxatg/aws-fpga-verify:20200131** already hosted at the Docker Hub.

### Prerequisites

To host a docker image, you need some sort of service. You can host it locally if you like (please read online docker instructions for that). However, this instruction uses [Docker Hub](https://hub.docker.com/)  as the hosting service.

Go to [https://hub.docker.com/signup](https://hub.docker.com/signup) to create a Docker Hub account (if you do not have one already), and then create a docker repository.

In this document, we are using an example docker account named as **memo40k**  and an example repository named as **k8s.** Please substitute these by your account and repository names respectively. Also, please note that you can set your repository as private if you do not want others to see it.

### Prepare docker images

####   
Step 1: Login to your Docker Hub account

`#docker login -u <username> -p <password>`

#### Step 2: Create a docker file

Here we will use our github folder [**docker/build_fpga_server_docker**](https://github.com/Xilinx/FPGA_as_a_Service/tree/master/k8s-fpga-device-plugin/trunk/docker/build_fpga_server_docker)  as an example. In this folder, **"server"** is a file folder that is to be added into our docker image. It has four files:

|File | Description|
|---|---|
| [fpga_algo.awsxclbin](https://github.com/Xilinx/FPGA_as_a_Service/blob/master/k8s-fpga-device-plugin/trunk/docker/build_fpga_server_docker/server/fpga_algo.awsxclbin "fpga_algo.awsxclbin")| This is the xclbin of the algorithm implemented on FPGA.|
| [fpga_host_exe](https://github.com/Xilinx/FPGA_as_a_Service/blob/master/k8s-fpga-device-plugin/trunk/docker/build_fpga_server_docker/server/fpga_host_exe "fpga_host_exe") | This is the host executable that downloads the xclbin to FPGA and interacts with the FPGA. |
|  [fpga_server.py](https://github.com/Xilinx/FPGA_as_a_Service/blob/master/k8s-fpga-device-plugin/trunk/docker/build_fpga_server_docker/server/fpga_server.py "fpga_server.py")|This is a representative server program that calls the host executable and has ability to receive command from a client. One can merge this with host executable into one single server program in C++.|
|   [run.sh](https://github.com/Xilinx/FPGA_as_a_Service/blob/master/k8s-fpga-device-plugin/trunk/docker/build_fpga_server_docker/server/run.sh "run.sh")  |  This sets environment and calls  | fpga_server.py.

You can add any number of folders with any contents you need for your server to work.

The **xilinxatg/aws-fpga-verify:20200131**  on docker hub is the base image as mentioned earlier. In this example, the folder  **server**  will be added to an example location **/opt/xilinx/k8s/** in the docker image.

`#touch Dockerfile  `

create a dockerfile under the same folder with server

`#vi Dockerfile  `

To add following two lines into Dockerfile

```
FROM xilinxatg/aws-fpga-verify:20200131  
COPY docker /opt/xilinx/k8s/server
```
#### Step 3: Build new docker image

`#docker build -t memo40k/k8s:accelator_pod .  `

It will build a new docker image called  **accelerator_pod**  using the docker file "**Dockerfile**" under the current folder

`#docker images  `

You can run this command to check whether the new images  **accelerator_pod**  was created.

`# docker run -it <imageID>`  

To test the docker image you just created, run the above. You should see the folder  **server** added into the docker image.

#### Step 4: Push new image into docker hub

`#docker push memo40k/k8s:accelator_podk8`




#### Step 5: Create a docker image for client

Please repeat the steps 2 to 4 with your desired executable contents for the client to create another docker image called  **test_client_pod**. You can use [build_test_client_docker](https://github.com/Xilinx/FPGA_as_a_Service/tree/master/k8s-fpga-device-plugin/trunk/docker/build_test_client_docker)  as an example.



### Verify docker image

Use the yaml files: [aws-accelator-pod.yaml](https://github.com/Xilinx/FPGA_as_a_Service/blob/master/k8s-fpga-device-plugin/trunk/aws-accelator-pod.yaml)  and [aws-test-client-pod.yaml](https://github.com/Xilinx/FPGA_as_a_Service/blob/master/k8s-fpga-device-plugin/trunk/aws-test-client-pod.yaml)  to create accelerator and client pods respectively.



#### Step 1: Create accelerator and client pods

`#kubectl create -f aws-accelator-pod.yaml  `

`#kubectl create -f aws-test-client-pod.yaml`

#### Step 2: Check pod status

After creating the two pods, there will be an accelerator pod with FPGA access, a client pod without FPGA access, an accelerator pod deployment service and a fpga-server-svc network service as shown below.

`#kubectl get pod`

```
NAME                          READY      STATUS    RESTARTS    AGE  
accelator-pod-ff67ff8b8-mwff   1/1       Running       0       22h  
test-client-pod                1/1       Running       0       23h
```

`#kubectl get deployment`

```
NAME             READY   UP-TO-DATE   AVAILABLE    AGE  
accelator-pod     1/1        1            1        22h
```

`#kubectl get service`

```
NAME               TYPE       CLUSTER-IP     EXTERNAL-IP    PORT(S)       AGE  
fpga-server-svc  NodePort     10.96.59.3        <none>   8010:31600/TCP   22h  
kubernetes       ClusterIP     10.96.0.1        <none>      443/TCP       14d
```

#### Step 3: Run hello world in client pod

`#kubectl exec test-client-pod python /opt/xilinx/k8s/client/client.py`



**Note:**

**If the status of the accelerator pod shows as pending, please check whether the card is already assigned to another running pod. If so, please delete the running pod and recreate the accelerator pod.**

