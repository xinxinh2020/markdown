[TOC]

# 部署安装



## Jupyter安装

```shell
# 安装python3

# 安装conda
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh

# 安装notebook
conda install -c conda-forge notebook
jupyter notebook --allow-root --ip 172.16.108.133 # 运行jupyter,启动notebook服务器，默认监听的ip地址是localhost
# 按照提示用浏览器打开
```



## 使用Docker部署用于测试的JupyterHub

```shell
# 不推荐在生产上使用这种方式部署
# 1 安装Docker 1.12以上版本

# 2 生成自签名的ssl证书
# openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout mykey.key -out mycert.pem
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout jupyterhub.key -out jupyterhub.crt
mkdir -p secrets
cp jupyterhub.crt jupyterhub.key secrets/

# 3 创建GitHub OAuth APP

# 4 把GitHub OAuth Client ID, Client Secret and OAuth callback url传给jupyterhub运行时
cd secrets/
vim oauth.env
cat oauth.env
GITHUB_CLIENT_ID=5baa9ff5d43d136482c3
GITHUB_CLIENT_SECRET=6f0a066ee230c708446aab12552673d67351fa2c
OAUTH_CALLBACK_URL=https://47.113.94.144/hub/oauth_callback


```





# 常用命令

```shell
jupyter notebook # 启动一个notebook服务器
                 # --help 查看帮助
                 # --no-browser 创建完notebook服务器后不要用浏览器打开
                 # --port 指定端口
                 # --ip 指定监听的地址
```





# 知识点

`ipython`是一个`python`的交互式`shell`，比默认的`python shell`好用得多。作为Jupiter notebook的一个REPL

REPL：Read-Eval-Print-Loop，提示用户输入代码，用户输入后，执行该代码，循环该过程

IPython Kernel是一个独立的进程用于运行用户的代码，以及计算可能完成的任务。notebook或qt console等前端通过Json消息与IPython内核通信，Json信息通过ZeroMQ socket发送。一个内核进程可以让多个前端连接，

Jupyter Notebook及其灵活的接口将Notebook从代码扩展到可视化、多媒体、协作等等。除了运行代码之外，它还将代码和输出以及markdown注释存储在一个称为notebook的可编辑文档中，当保存这个文档时，它会被发送到notebook服务器上，服务器会把它保存为一个.ipynb的json文件。注意是notebook server负责保存和加载notebooks，而不是ipython内核，内核只是简单地获取代码并执行。jupyter的Nbconvert工具可以把notebook转换成其它格式，如xml。



## 架构图

![Architecture diagram of project relationships](https://jupyter.readthedocs.io/en/latest/_images/repos_map.png)

