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

## 集群连接，NCCL配置
Network Installer for Ubuntu22.04
````
$ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.0-1_all.deb
$ sudo dpkg -i cuda-keyring_1.0-1_all.deb
$ sudo apt-get update
Network Installer for Ubuntu20.04

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

For Ubuntu: sudo apt install libnccl2=2.13.4-1+cuda11.7 libnccl-dev=2.13.4-1+cuda11.7
For RHEL/Centos: sudo yum install libnccl-2.13.4-1+cuda11.7 libnccl-devel-2.13.4-1+cuda11.7 libnccl-static-2.13.4-1+cuda11.7

##NCCL-TEST
sudo apt install openmpi-bin

cd /opt
mkdir nccl-test
cd nccl-test
git clone https://github.com/NVIDIA/nccl-tests.git
cd nccl-test

make
````

````
如果出现libc6错误.
在 Ubuntu 20.04 中，libc6 2.31 是默认安装的 libc6 版本。要升级 libc6 到一个更高的版本，你需要添加一个源，并执行相应的升级命令。

请注意，升级 libc6 是一个敏感的操作，可能会对系统产生严重影响，包括导致系统不稳定或不可用。在执行下面的步骤之前，请确保你了解潜在的风险，并在进行更改之前备份重要的数据和系统配置。

下面是一个可能的方法来升级 libc6：

编辑源列表文件：打开终端，并使用管理员权限编辑 /etc/apt/sources.list 文件。可以使用以下命令来编辑文件：

shell
Copy code
sudo nano /etc/apt/sources.list
添加更新源：在文件的末尾添加以下两行来添加更新源：

shell
Copy code
deb http://archive.ubuntu.com/ubuntu/ focal-proposed restricted main multiverse universe
deb-src http://archive.ubuntu.com/ubuntu/ focal-proposed restricted main multiverse universe
注意：focal-proposed 是针对 Ubuntu 20.04（代号为 Focal Fossa）的，你可以根据你的 Ubuntu 版本进行调整。

保存并关闭文件：按下 Ctrl + X 键，然后输入 Y 保存文件并退出编辑器。

更新软件包列表：运行以下命令来更新软件包列表：

shell
Copy code
sudo apt-get update
升级 libc6：运行以下命令来升级 libc6：

shell
Copy code
sudo apt-get install libc6
这将尝试升级 libc6 到可用的最新版本。

移除更新源：升级完成后，可以将更新源从 /etc/apt/sources.list 文件中移除，以避免后续使用出现问题。使用管理员权限打开 /etc/apt/sources.list 文件，并删除添加的更新源行，然后保存文件。

更新软件包：最后，运行以下命令来确保系统中的所有软件包都是最新的：

shell
Copy code
sudo apt-get update
sudo apt-get upgrade
请注意，这只是一种尝试升级 libc6 的方法，并且可能因系统配置、依赖关系和其他因素而有所不同。如果你遇到问题或不确定如何处理，建议寻求专业支持或咨询相关的技术资源。
````






