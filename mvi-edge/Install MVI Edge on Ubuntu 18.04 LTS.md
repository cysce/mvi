# Install Maximo Visual Inspection Edge v1.3

## Part 1 - Hardware & Software Requeriments

### Hardware

#### Disk space requirements
* Installation - The IBM Maximo Visual Inspection Edge install package contains Docker containers for deployment on all supported platforms and requires 15 GB to download. Only the images needed for the platform will be installed by the load_images.sh operation, but this requires at least 70 GB available in the file system used by Docker, usually /var/lib/docker.
* Deploying a model - Models are extracted into the /tmp directory before loading. The size of the model depends on the framework, but at least 1 GB should be available in /tmp before deploying a model.

#### GPU model requirements
* IBM Maximo Visual Inspection Edge is supported only on NVIDIA Tesla GPUs: T4, V100, and P100.

#### GPU memory requirements
* For deployment, the amount of memory required depends on the type of model you want to deploy. To determine how large a deployed model is, run nvidia-smi from the host after deployment. Find the corresponding PID that correlates to the model you deployed and look at the Memory Usage.
* A custom model based on TensorFlow will take all remaining memory on a GPU. However, you can deploy it to a GPU that has at least 2 GB memory.

### Platform Requirements

* IBM Maximo Visual Inspection Edge can be deployed on x86 and IBM Power Systems platforms.
* YOLO v3, Detectron, and SSD models require Nvidia GPUs. Other models can be deployed in CPU only environments.

### Software

#### Linux
* Red Hat Enterprise Linux (RHEL) RHEL 7.6 ALT (little endian) for POWER9
* RHEL 7.7 for x86 and POWER8.
* Ubuntu 18.04

Note: The Ubuntu Hardware Enablement (HWE) kernel is not supported. Kubernetes services do not function correctly, preventing IBM Maximo Visual Inspection from starting successfully.

#### NVIDIA CUDA
* x86 - 10.1 or later drivers. For information, see the NVIDIA CUDA Toolkit website.
* ppc64le - 10.2 or later drivers are required. For information, see the NVIDIA CUDA Toolkit website.

#### Docker
* RHEL: docker-1.13.1
* Ubuntu: Docker CE or EE 18.06.01
* When running Docker, nvidia-docker 2 is supported. For RHEL 7.6, see Using nvidia-docker 2.0 with RHEL 7.

#### Unzip
* The unzip package is required on the system to deploy the zipped models.

## Part 2 - Installing Docker and NVDIA Docker 2

### Install Docker (Ubuntu)
Use these steps to install Docker.

#### IBM Power
```
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-
properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=ppc64el] https://download.docker.com/linux/ubuntu
bionic stable"
sudo apt-get update
sudo apt-get install docker-ce=18.06.1~ce~3-0~ubuntu
```

#### x86_64
```
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-
properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu
bionic stable"
sudo apt-get update
sudo apt-get install docker-ce
```

### Install nvidia-docker2 (Ubuntu)
Use these steps to install nvidia-docker 2.

```
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo
tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
sudo apt-get install nvidia-docker2
sudo systemctl restart docker.service
```

### Verify the Setup

#### IBM Power
```
docker run --rm nvidia/cuda-ppc64le:10.2-base-ubuntu18.04 nvidia-smi
```

#### x86_64
```
docker run --rm nvidia/cuda nvidia-smi
```

## Part 3 - Installing Maximo Visual Inspection Edge Version 1.3.0

Decompress the product tar file, change directories to the newly created directory, then run the installation command. For example, using the downloaded package for Power:

```
$ tar -xvf vis-inspct-edge-ppc-1.3.0-ppa.tar
vis-inspct-edge-ppc-1.3.0-ppa/
vis-inspct-edge-ppc-1.3.0-ppa/visual-inspection-edge-ppc64le-containers-1.3.0.0.tar
vis-inspct-edge-ppc-1.3.0-ppa/visual-inspection-edge-1.3.0.0-472.22b7a1d.ppc64le.rpm
vis-inspct-edge-ppc-1.3.0-ppa/visual-inspection-edge_1.3.0.0-472.22b7a1d_ppc64el.deb
$ cd vis-inspct-edge-ppc-1.3.0
```

Run the installation command for the platform you are installing on:
### RHEL
```
sudo yum install ./<file_name>.rpm 
```
### Ubuntu
```
sudo dpkg -i ./<file_name>.deb
```

Load the product Docker images with the container tar file:
```
/opt/ibm/vision-edge/bin/load_images.sh -f <tar_file>
```
The file name has this format: visual-inspection-edge-\<arch\>-containers-\<release
\>.tar, where \<arch\> is x86 or ppc, and \<release\> is the product version being installed. 

IBM Maximo Visual Inspection Edge will be installed at /opt/ibm/vision-edge.

## Part 4 - Uninstalling IBM Maximo Visual Inspection Edge
Follow these steps to uninstall IBM Maximo Visual Inspection Edge:
1. It is recommended that deployed models be undeployed.
2. To recover disk space, run docker rmi to remove the "deploy" docker images with the current release tag. For example, to remove all deploy images for the 1.3.0.0 release:
   ```
   $ for deploy_img in $(docker images --format '{{.Repository}}:{{.Tag}}' | egrep 'vision-dnn- [a-z]*[-]*deploy[-]*[a-z0-9]*\:1.3.0.0') ; do docker rmi ${deploy_img} ; echo "Purged image $ {deploy_img}." ; done
   ```
3. Optionally also uninstall the package: 
   
   • For RHEL:
   ```
   sudo yum remove visual-inspection-edge
   ```

   • For Ubuntu:

   ```
   sudo dpkg --remove visual-inspection-edge
   ```