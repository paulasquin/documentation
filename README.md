# Documentation
by Paul Asquin. 
Tips and documentation to save time when problems occur twice

## Machine Learning
### CUDA
Setting cuda up should be done with ubuntu repo. Nvidia binary solutions fail too often.
Keep it simple
```
```

### Jupyter
- set a password
```bash jupyter notebook password
```
then open an instance and

## Server
### UFW
Ubuntu easy to set firewall. [Introduction here](https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server)  
**Warning**: when setting up `ufw`, do not forget to allow immediately ssh connection, then disable, then re-enable ufw to apply changes. Otherwise, you'll get lock outside of ubuntu server you would be working on.

## Docker

### GPU
- Set Nvida Docker runtime
[Official tuto is good](https://github.com/NVIDIA/nvidia-container-runtime). I didn't followed the nvidia-docker2 steps, I got issue getting it from the additional ppa, and I decided to manually set the docker runtime config as presented in the steps.

- Test GPU is working
```bash
docker run --runtime=nvidia --rm nvidia/cuda:10.1-cudnn7-devel-ubuntu18.04 nvidia-smi
```
