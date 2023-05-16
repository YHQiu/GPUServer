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
apt install ubuntu-drivers-common
````

3、自动配置驱动并重启服务器
````
ubuntu-drivers autoinstall
reboot
````
4、安装Aconda
````
apt install anaconda;source ~/.bashrc;conda --version
````
5、进入conda环境
````
### conda activate <environment-name>
### <environment-name> 替换为你的环境名称，如env_1
conda activate env_1
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
nohup python3 -m fastchat.serve.model_worker --model-path lmsys/fastchat-t5-3b-v1.0 --controller http://localhost:21001 --port 31000 --worker http://localhost:31000 --gpus 1 > fschat_model_output.log &
````
#访问web接口测试是否Run成功
http://192.168.1.212:21001

#查询运行的服务
ps -ef |grep fastchat
#杀掉运行的服务
kill -9 <PID>
