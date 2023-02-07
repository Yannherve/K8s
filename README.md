#Install network tools
apt-get install net-tools
apt-get install jq -y

# Enable kernel modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Add some settings to sysctl
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Reload sysctl
sudo sysctl --system


echo "Installing docker"
apt-get update
apt-get install -y \
        apt-transport-https \
        ca-certificates \
        curl \
        software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository \
        "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
        $(lsb_release -cs) \
        stable"

apt-get update && apt-get install -y \
        docker-ce=$(apt-cache madison docker-ce | grep 20.10 | head -1 | awk '{print $3}')


# Create required directories
sudo mkdir -p /etc/systemd/system/docker.service.d

# Create daemon json config file
sudo tee /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

# Start and enable Services
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl enable docker



echo "Installing Kubernetes"
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
#Kubernets version 1.24.0 is working properly, downgrading to 1.23.0
apt-get install -qy kubelet=1.23.0-00 kubeadm=1.23.0-00 kubectl=1.23.0-00 --allow-downgrades



sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapoff -a

sudo systemctl enable kubelet

NB1: If when running the kubeadm reset/init and having the error, just vi /etc/docker/daemon.json then paste this
{
    "exec-opts": ["native.cgroupdriver=systemd"]
}
inside


NB2: sudo apt-get install -y kubelet=1.23.0-00 kubeadm=1.23.0-00 kubectl=1.23.0-00 to downgrade the current version because of his stability.
