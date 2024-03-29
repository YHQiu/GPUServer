****教程目录****


## 1、A100 GPU 服务驱动、环境配置安装教程

0.Pre前置步骤，网络配置：

#切换到root环境，方便后续配置操作
````
su root
````
#如果没有配置root密码，则passwd root添加新密码，再su root

#查看主网卡
````
ifconfig
````
#主网卡一般是第一个

#配置网络
````
cd /etc/netplan
ls 
````

#vi <你看到的文件名>
如
````
vi 00-installer-config.yaml
````

#找到你的网卡添加代码,将eno1替换为你的网卡名，修改IP 网关和域名解析地址：

````
eno1:
   dhcp4: no
   addresses: [192.168.1.212/24]
   gateway4: 192.168.1.1
   nameservers:
     addresses: [114.114.114.114,8.8.8.8, 8.8.4.4]
````

#配置完成后按esc键，再输入如下代码

````
:wq
````

#最后执行：
````
sudo netplan apply
````
#使配置生效

1、切换到ROOT环境并切换APT源
````
su root
sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
apt update
````

2、安装Ubuntu-drivers
````
apt install alsa-utils
apt install ubuntu-drivers-common
````

3、自动配置驱动并重启服务器
````
ubuntu-drivers autoinstall
reboot
````
4、安装Aconda

````
### 镜像安装法
cd /opt;mkdir anaconda;cd anaconda;wget https://repo.anaconda.com/archive/Anaconda3-2022.05-Linux-x86_64.sh
bash Anaconda3-2022.05-Linux-x86_64.sh
echo 'export PATH="$HOME/anaconda3/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes
conda info
````
5、进入conda环境
````
### conda activate <environment-name>
### <environment-name> 替换为你的环境名称，如env_1
conda activate env_1
### 注意进入conda环境以后您使用的Python pip等被替换为conda环境的独立包。要退出该环境回到系统环境，可以执行
conda deactivate
### 要查看conda环境，可以执行
conda env list
````

6、测试环境，这里我们使用FastChat执行推理来测试GPU服务器配置情况。

#使用conda 安装 cuda（简化的方案，正式服务器此步骤可能需要物理安装）
````
conda install -c anaconda cudatoolkit
````
#在物理环境安装CUDA(推荐)
#首先在Nidia官方找到CUDA下载链接 官网地址<https://developer.nvidia.com/cuda-toolkit-archive>
#然后找到一个版本，比如11.7
````bash
cd /opt;mkdir cuda_shell
wget https://developer.download.nvidia.com/compute/cuda/11.7.1/local_installers/cuda_11.7.1_515.65.01_linux.run
sudo sh cuda_11.7.1_515.65.01_linux.run
# 安装提示安装，此时会弹出CUDA选配项，按自己的需求安装。
````

#创建一个文件夹

````
mkdir -r  /usr/local/apps/fastchat;cd /usr/local/apps/fastchat
````

#安装fschat

````
conda install fschat
````

#启动fschat controller 模型管理控制器

````
nohup python3 -m fastchat.serve.controller > fschat_controller_output.log &
````

#启动fachat web 服务
````
nohup python3 -m fastchat.serve.gradio_web_server > fschat_web_output.log &
````

#启动API服务
````
nohup python3 -m fastchat.serve.openai_api_server > fschat_api_output.log &
````

#使用GPU执行推理，测试环境
````
nohup python3 -m fastchat.serve.model_worker --model-path lmsys/fastchat-t5-3b-v1.0 --controller http://localhost:21001 --port 31000 --worker http://localhost:31000 --nums-gpus 1 > fschat_model_output.log &
````
#访问web接口测试是否Run成功
http://192.168.1.212:21001

#查询运行的服务
ps -ef |grep fastchat
#杀掉运行的服务
kill -9 <PID>

## 集群连接，NCCL配置，以及NCCL-TEST

*安装NCCL*
````
#根据系统版本选择一个安装即可。

Network Installer for Ubuntu20.04
##20系统用这个
$ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-keyring_1.0-1_all.deb
$ sudo dpkg -i cuda-keyring_1.0-1_all.deb
$ sudo apt-get update

Network Installer for Ubuntu18.04
$ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-keyring_1.0-1_all.deb
$ sudo dpkg -i cuda-keyring_1.0-1_all.deb
$ sudo apt-get update

Network Installer for RedHat/CentOS 8
$ sudo dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel8/x86_64/cuda-rhel8.repo

Network Installer for RedHat/CentOS 7
$ sudo yum-config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/cuda-rhel7.repo

For Ubuntu:
````bash
sudo apt install libnccl2=2.13.4-1+cuda11.7 libnccl-dev=2.13.4-1+cuda11.7
````
For RHEL/Centos: 
````bash
sudo yum install libnccl-2.13.4-1+cuda11.7 libnccl-devel-2.13.4-1+cuda11.7 libnccl-static-2.13.4-1+cuda11.7
````

*安装NCCL-TEST*
````bash
sudo apt install openmpi-bin
cd /opt
mkdir nccl-test
cd nccl-test
git clone https://github.com/NVIDIA/nccl-tests.git
cd nccl-test

sudo apt-get install libopenmpi-dev
sudo mkdir /usr/include/mpi/
sudo ln -s /usr/lib/x86_64-linux-gnu/openmpi/include/* /usr/include/mpi/

````

*构建NCCL-TEST*
````bash
# 设置环境变量
MPI=/usr/lib/x86_64-linux-gnu/openmpi/
NCCL=/usr/include/
CUDA=/usr/cuda
# 构建（二选一）
# 单机
make
OR
# 集群
make MPI=1 MPI_HOME=/usr/lib/x86_64-linux-gnu/openmpi/ CUDA_HOME=/usr/local/cuda NCCL_HOME=/ucr/include
````

*测试NCCL*
````bash
CUDA_DEVICE=0,1  ./build/all_reduce_perf -b 8 -e 128M -f 2 -g 2
````










