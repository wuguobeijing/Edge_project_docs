## 安装
**关闭交换空间**
sudo swapoff -a
**关闭开机自启动**
sudo sed -i "/ swap/ s/^/#/ " /etc/fstab

### 下载
      git clone https://github.com/kubernetes/kubernetes
      cd k8s/kubernetes
      make quick-release
### 在 Linux 系统中安装 kubectl
