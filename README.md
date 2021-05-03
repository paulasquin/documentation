# Documentation
by Paul Asquin. 
Tips and documentation to save time when problems occur twice

## Machine Learning
### CUDA & CUDNN
Setting cuda up should be done with ubuntu repo. Nvidia binary solutions fail too often.
Keep it simple
- clean
```bash
sudo rm /etc/apt/sources.list.d/cuda*
sudo apt remove -y --autoremove nvidia-*
sudo apt-get autoremove -y
sudo apt-get autoclean -y
```
- prepare, install
```bash
sudo apt update
sudo add-apt-repository ppa:graphics-drivers

# Ubuntu 18.04
sudo apt-key adv --fetch-keys  http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda.list'
sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda_learn.list'
sudo apt update
sudo apt install -y cuda-10-1
sudo apt install -y libcudnn7

# Ubuntu 20.04
sudo apt-key adv --fetch-keys  https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64 /" > /etc/apt/sources.list.d/cuda.list'
sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu2004/x86_64 /" > /etc/apt/sources.list.d/cuda_learn.list'
sudo apt update
sudo apt install cuda-11-3
sudo apt install libcudnn8
```
- and check
```bash
sudo nvidia-smi
```
And it works in a "from scratch" ubuntu installation.  
No Cudnn subscription, no binary scripts always failing for dark reasons, no "nouveau" module disabling... 
This is clean, simple and *saves time!*

### Jupyter
- set a password
```bash 
jupyter notebook password
```

## Server
### UFW
Ubuntu easy to set firewall. [Introduction here](https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server)  
**Warning**: when setting up `ufw`, do not forget to allow immediately ssh connection, then disable, then re-enable ufw to apply changes. 
Otherwise, you'll get lock outside of ubuntu server you would be working on.  

Example:
```bash
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow 8888
```
and to update, never forget
```bash
sudo ufw reload
```

Note: ufw gets invisible to nmap as it deactivate host discovery.
It's possible to see the true mapping with 
```bash
nmap IP -Pn
```

## Docker

### GPU
- Set Nvida Docker runtime
[Official tuto is good](https://github.com/NVIDIA/nvidia-container-runtime). I didn't followed the nvidia-docker2 steps, I got issue getting it from the additional ppa, and I decided to manually set the docker runtime config as presented in the steps.

```bash
curl -s -L https://nvidia.github.io/nvidia-container-runtime/gpgkey | sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-container-runtime/$distribution/nvidia-container-runtime.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-runtime.list
sudo apt-get update
sudo apt-get install -y nvidia-container-runtime
```

- Test GPU is working
```bash
docker run --runtime=nvidia --rm nvidia/cuda:10.1-cudnn7-devel-ubuntu18.04 nvidia-smi
```

### Docker Compose CUDA
End of the Machine Learning setting journey. Use CUDA + Docker + Docker compose. 
With all the three installed, we need to use as docker-compose base:

```yaml
version: '2.3'

services:
  your_server:
    build:
      context: .
      dockerfile: Dockerfile.gpu
    container_name: your_name
    runtime: nvidia
    environment:
      - NVIDIA_VISIBLE_DEVICES=0
```
assuming you are using Dockerfile.gpu and that you want to use GPU#0 only.  
Docker Compose v3.0 doesn't work for now with nvidia runtime

