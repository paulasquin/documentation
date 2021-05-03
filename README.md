# Documentation
by Paul Asquin. 
Tips and documentation to save time when problems occur twice

## Docker
### Install Docker
- From [official documentation](https://docs.docker.com/engine/install/ubuntu/)
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
 sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io
```

- Then you can provide sudo rights to docker by default with
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```
You'll need to disconnect then reconnect from your ubuntu account

- Check the installation with 
```bash
docker run hello-world
```

### Install Docker Compose
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

- Check the installation with 
```bash
docker-compose --version
```

If if fails, you can try
```bash
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```


## Machine Learning
### Install CUDA & CUDNN
Setting cuda up should be done with ubuntu repo. Nvidia binary solutions fail too often.
Keep it simple

- Clean from previously installed drivers
```bash
sudo rm /etc/apt/sources.list.d/cuda*
sudo apt remove -y --autoremove nvidia-*
sudo apt-get autoremove -y
sudo apt-get autoclean -y
```
- Get PPA
```bash
sudo apt update
sudo add-apt-repository ppa:graphics-drivers
```

- Install for **Ubuntu 18.04**
```bash
sudo apt-key adv --fetch-keys  http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda.list'
sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda_learn.list'
sudo apt update
sudo apt install -y cuda-10-1
sudo apt install -y libcudnn7
```

- Install for **Ubuntu 20.04**
```bash
sudo apt-key adv --fetch-keys  https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64 /" > /etc/apt/sources.list.d/cuda.list'
sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu2004/x86_64 /" > /etc/apt/sources.list.d/cuda_learn.list'
sudo apt update
sudo apt install cuda-11-3
sudo apt install libcudnn8
```

- Reboot your computer, then check
```bash
sudo nvidia-smi
```
This works directly with a "from scratch" ubuntu installation.  
No Cudnn subscription, no binary scripts always failing for dark reasons, no "nouveau" module disabling... 
This is clean, simple and *saves time!*

### Make Docker works with GPU
[Official tuto is good](https://github.com/NVIDIA/nvidia-container-runtime). I didn't followed the nvidia-docker2 steps, I got issue getting it from the additional ppa, and I decided to manually set the docker runtime config as presented in the steps.

- Get PPA
```bash
curl -s -L https://nvidia.github.io/nvidia-container-runtime/gpgkey | sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-container-runtime/$distribution/nvidia-container-runtime.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-runtime.list
```

- Install
```bash
sudo apt-get update
sudo apt-get install -y nvidia-container-runtime
```

- Configure the runtime
```bash
sudo tee /etc/docker/daemon.json <<EOF
{
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
EOF
sudo pkill -SIGHUP dockerd
```

- Test GPU is working
```bash
docker run --runtime=nvidia --rm nvidia/cuda:10.1-cudnn7-devel-ubuntu18.04 nvidia-smi
```

### Use Docker GPU through Compose
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


## Jupyter
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




