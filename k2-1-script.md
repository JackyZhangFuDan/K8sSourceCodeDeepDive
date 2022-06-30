# 安装 Kubernetes 开发环境的脚本

在 [Kubernetes开发之旅 第一阶段](https://www.bilibili.com/video/BV11U4y1y7V7/)，我系统介绍了如何在Ubuntu 20.04 Desktop版本中安装Kubernetes 1.24.0的开发环境，这篇文档给出其中用到的脚本命令，不适合直接阅读，请结合以上视频使用。

## 为Ubuntu换阿里源
```sh
sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak  
sudo nano /etc/apt/sources.list  
```
```sh
用如下阿里源替换该文件的已有内容：  
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse  
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse  
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse  
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse  
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse  
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse  
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse  
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse  
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse  
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse  

sudo apt-get update  
```

## 安装GNU  
sudo apt install build-essential  

## 安装Docker
sudo apt-get update  
sudo apt-get install \  
    ca-certificates \  
    curl \  
    gnupg \  
    lsb-release  

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg  

echo \  
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \  
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null  

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

## 修改ContainerD 所用的镜像库地址  
containerd config default > ~/config.toml  
然后编辑～config.toml去添加信息，具体内容请看视频中这个环节
sudo mv ~/config.toml /etc/containerd/config.toml  
sudo systemctl restart containerd  

## 安装rsync:
cd ~/Downloads  
wget https://github.com/WayneD/rsync/archive/refs/tags/v3.2.4.tar.gz  
tar -xf v3.2.4.tar.gz  
cd rsync-3.2.4  
安装一些工具包  
sudo apt install -y gcc g++ gawk autoconf automake python3-cmarkgfm  
sudo apt install -y acl libacl1-dev  
sudo apt install -y attr libattr1-dev  
sudo apt install -y libxxhash-dev  
sudo apt install -y libzstd-dev  
sudo apt install -y liblz4-dev  
sudo apt install -y libssl-dev  
编译，安装  
 ./configure  
 make  
sudo cp ./rsync /usr/local/bin/  
sudo cp ./rsync-ssl /usr/local/bin/  

## 安装jq：  
sudo apt-get install jq  

## 安装pyyaml:  
sudo apt install python3-pip  
pip install pyyaml  

## 安装etcd：  
ETCD_VER=v3.5.4  
curl -L https://storage.googleapis.com/etcd/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz  
mkdir ~/etcd  
tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C ~/etcd --strip-components=1  
rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz  

sudo nano ~/.bashrc  
最后加入：export PATH="/home/<用户名>/etcd:${PATH}"  
source ~/.bashrc  

## 安装golang (1.24及以上需要golang 1.18)：  
cd ~/Downloads  
wget https://golang.google.cn/dl/go1.18.2.linux-amd64.tar.gz  
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.18.2.linux-amd64.tar.gz  

mkdir ~/go  
mkdir ~/go/src  
mkdir ~/go/bin  
sudo nano ~/.bashrc  

最后加入如下几行：  
export GOPATH="/home/<用户名>/go"  
export GOBIN="/home/<用户名>/go/bin"  
export PATH="/usr/local/go/bin:$GOPATH/bin:${PATH}"  

source ~/.bashrc  

sudo nano /etc/sudoers  
在secure_path一行加入如下目录：  
/usr/local/go/bin （这个是$GOPATH/bin目录）  
/home/<用户名>/etcd （这个是etcd命令所在目录）  
/home/<用户名>/go/bin （这个是go get安装的程序所在位置）  

### 设置golang代理：  
go env -w GO111MODULE="on"  
go env -w GOPROXY="https://goproxy.cn,direct"  

## 安装CFSSL：  
go install github.com/cloudflare/cfssl/cmd/...@latest  

## 下载kubernetes代码：  
mkdir $GOPATH/src/k8s.io  
git clone https://github.com/kubernetes/kubernetes.git  
git checkout -b kube1.24 v1.24.0  

## 编译启动本地单节点集群：  
cd $GOPATH/src/k8s/kubernetes
编译单个组建：sudo make WHAT="cmd/kube-apiserver"  
编译所有组件：sudo make all  
启动本地单节点集群： sudo ./hack/local-up-cluster.sh  
