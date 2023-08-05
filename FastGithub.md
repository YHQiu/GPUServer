## 1、FataGithub安装
````bash
从
https://github.com/dotnetcore/fastgithub/releases
找到对应的下载链接。

下载完成上传到服务器，unzip 文件
cd 进入文件
sudo ./fastgithub start // 以systemd服务安装并启动
sudo ./fastgithub stop // 以systemd服务卸载并删除
````

## 2、Github代理
````bash
# 找github https clone地址
# git clone https://ghproxy.com/{your github clone address}
# 如下,比如要克隆/shap-e仓库则按如下方式进行
git clone https://ghproxy.com/https.github.com/https://github.com/openai/shap-e.git
````
