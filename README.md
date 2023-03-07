# Documentation

by Paul Asquin.
Tips and documentation to save time when problems occur twice

# Machine Setup

## Install Docker

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

## Install Docker Compose

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

## Install CUDA & CUDNN

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
sudo apt install -y cuda-10-1 libcudnn7
```

- Install for **Ubuntu 20.04**

```bash
sudo apt-key adv --fetch-keys  https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64 /" > /etc/apt/sources.list.d/cuda.list'
sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu2004/x86_64 /" > /etc/apt/sources.list.d/cuda_learn.list'
sudo apt update
sudo apt install -y cuda-11-3 libcudnn8
```

- Reboot your computer, then check

```bash
sudo nvidia-smi
```

This works directly with a "from scratch" ubuntu installation.  
No Cudnn subscription, no binary scripts always failing for dark reasons, no "nouveau" module disabling...
This is clean, simple and _saves time!_

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

## Use Docker GPU through Compose

End of the Machine Learning setting journey. Use CUDA + Docker + Docker compose.
With all the three installed, we need to use as docker-compose base:

```yaml
version: "2.3"

services:
  your_service:
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

# Windows Machine Setup

It will always be easier to work with Docker, Docker Compose and Docker GPU from a Ubuntu machine,
still one of the great strength of Docker is to allow you to deploye your projets in any machine.

You can totally install Docker within windows, and it will come direclty with Docker-Compose.
Still, if you want to use GPU capacities + Docker + Windows host machine, you will need to do this through Windows Subsystem for Linux.
You will find the detailled installation process [here](https://docs.nvidia.com/cuda/wsl-user-guide/index.html) leading you throught the installation
of the Ubuntu subsystem, Docker, Cuda, and Nvidia-docker.

Note: when starting a docker image, you may need to use --gpus=all instead of runtime=nvidia.
Within a docker-compose.yml file, just use

```yaml
services:
  environment:
    - NVIDIA_VISIBLE_DEVICES=0`
# ensure runtime=nvidia is not set
```

# Jupyter

- set a password

```bash
jupyter notebook password
```

# Server

## UFW

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

# Pipeline Management

In order to track pipeline events and to monitor it from a distant machine, [Prefect](prefect.io) is a good candidate.

## Installation

Requirements:

- Docker
- Docker Compose
  Installation

```bash
pip install prefect
```

Terminology:

- An agent is a worker which will take a pipeline load. It can be a local instance, docker, kubernetes, EC2.
- A User Interface, it's where the user can monitor the pipeline state. It can be locally served our cloud served

### Init Cloud User Interface

- Create an account & API key on [Prefect](prefect.io) then authenticate

```bash
prefect auth login --key YOUR_KEY
```

- Switch to cloud prefect server

```bash
prefect backend cloud
```

## Init Local User Interface

You can also have a local user interface for Prefect. To do so

- Create a server

```bash
prefect server start
```

- Switch to local prefect server

```bash
prefect backend server
```

- Create tenant

```bash
prefect server create-tenant --name default --slug default
```

## Create a local agent

- Create project

```bash
prefect create project "YOUR_NAME"
```

- Create agent

```bash
prefect agent local start
```

## Define & Run a pipeline

- Define pipeline flow [link](https://docs.prefect.io/orchestration/getting-started/registering-and-running-a-flow.html#register-a-flow)
- Register it

```bash
python your_file.py
```

- Go to your [cloud dashboard](https://cloud.prefect.io/) or [local dashboard](http://localhost:8080) and run / schedule / monitor your pipeline
- You can run a pipeline direclty using the command line

```bash
prefect run -n "FLOW_NAME" --watch
```

with flow name as defined inside python file.

# Python Documentation

To generate python documentation on-the-go, pdoc is an amazing tool.  
Get it

```bash
pip install pdoc3
```

Generate a documentation server

```bash
pdoc --http localhost:4242 YOUR_MODULE
```

You can also get documentation in PDF or HTML

# Visual Code

Visual Code is quite good to develop in Python. Pycharm is good too, but a bit older in some sense.

- Run refacto with VSCode
```
[Option or Alt] + Maj + F
```

- Render a Markdown page
```
[Command or Control] + Maj + V
```

- Enter palette
```
[Command or Control] + Maj + P
```
Think about running "Install 'code' command in path"

- Open any project in VSCode
```
code .
```

# Docker Dev

To enter a local Dockerfile machine, run

```bash
docker run -it $(docker build -q .)
```

# Github SSH current bug
Sourced from this [gist](https://gist.github.com/Tamal/1cc77f88ef3e900aeae65f0e5e504794)  

Performing 
```bash
git clone ...
```
if you get error such as 
> fatal: Could not read from remote repository.  
> ssh: connect to host github.com port 22: Connection refused  

- Check that this is not linked to your access rights to a repo 
running the following line, which should lead to the same error
```bash
ssh -T git@github.com
```

- Ensure that using secured port 443 with ssh is working for this fix

```bash
ssh -T -p 443 git@ssh.github.com
```
> Hi xxxx! You've successfully authenticated, but GitHub does not provide shell access.

- Then, we can add specific rules for ssh github to always use the 443 port

Modify `vim ~/.ssh/config` and add the section
```
Host github.com
  Hostname ssh.github.com
  Port 443
```
- Finally, this command should work
```bash
ssh -T git@github.com
```

# VPN
OpenVPN configuration is more fine-tunnable with TunnelBlick.  
IP-V6 should be disabled from local when running a VPN to avoid any ip-v6 leaks 
