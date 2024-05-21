# Install Maximo Visual Inspection v1.3

## Part 1 - Hardware & Software Requeriments

### Hardware

#### Disk space requirements
IBM Maximo Visual Inspection has the following storage requirements for the initial product installation and for the data sets that will be managed by the product.
* /var - The product installation requires at least 75 GB of space in the /var file system for the product Docker images. IBM Maximo Visual Inspection also generates log information in this file system. The installation process requires additional space because all Docker images are extracted to disk requiring about 40 GB of space, then the images are loaded by Docker. When the image is loaded, the extracted image is deleted but while images are being extracted and loaded, space is needed for both copies. All application Docker images are extracted and loaded in parallel.

   Recommendation: If you want to minimize the root (/) file system, make sure that /var has its own volume.
* /opt - IBM Maximo Visual Inspection data sets, models, and runtime data are stored in this file system. This file system must have at least five GB of free space, in addition to any data sets, models or other runtime data for IBM Maximo Visual Inspection to operate successfully. The storage needs will vary depending on the data sets and the contents. For example, video data can require large amounts of storage.
   
   Recommendation: If you want to minimize the root (/) file system, make sure that /opt has its own volume. The /opt file system should have at least 40 GB of space, although this value might be more depending on your data sets.

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

#### x86_64
```
docker run --rm nvidia/cuda nvidia-smi
```

## Part 3 - Installing Maximo Visual Inspection Version 1.3.0

Decompress the product tar file, change directories to the newly created directory, then run the installation command. For example, using the downloaded package for Power:

```
$ tar -xvf visual-inspect-x86-1.3.0-ppa.tar
visual-inspect-x86-1.3.0-ppa/
visual-inspection-x86-1.3.0.0-ppa/visual-inspection-images-x86-1.3.0.0.tar
visual-inspection-x86-1.3.0.0-ppa/visual-inspection-1.3.0.0-552.07b6d7c.x86_64.rpm
visual-inspection-x86-1.3.0.0-ppa/visual-inspection_1.3.0.0-552.07b6d7c_amd64.deb
$ cd visual-inspection-x86-1.3.0.0-ppa
```

Run the installation command for the platform you are installing on:

### Ubuntu
```
sudo dpkg -i ./<file_name>.deb
```

```
sudo dpkg -i ./visual-inspection_1.3.0.0-552.07b6d7c_amd64.deb
```

Load the product Docker images with the container tar file:
```
/opt/ibm/vision/bin/load_images.sh -f <tar_file>
```
The file name has this format: visual-inspection-\<arch\>-containers-\<release
\>.tar, where \<arch\> is x86 or ppc, and \<release\> is the product version being installed. 

IBM Maximo Visual Inspection will be installed at /opt/ibm/vision.

## Part 4 - Uninstalling IBM Maximo Visual Inspection

Follow these steps to uninstall IBM Maximo Visual Inspection:
1. It is recommended that deployed models be undeployed.
2. To recover disk space, run docker rmi to remove the "deploy" docker images with the current release tag. For example, to remove all deploy images for the 1.3.0.0 release:
   ```
   $ for deploy_img in $(docker images --format '{{.Repository}}:{{.Tag}}' | egrep 'vision-dnn- [a-z]*[-]*deploy[-]*[a-z0-9]*\:1.3.0.0') ; do docker rmi ${deploy_img} ; echo "Purged image $ {deploy_img}." ; done
   ```
3. Optionally also uninstall the package: 
   
   • For Ubuntu:

   ```
   sudo dpkg --remove visual-inspection
   ```