Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-20.04"
  config.vm.box_version = "202112.19.0"
  config.vm.hostname = 'cka-cp'
  config.vm.define vm_name = 'cka-cp'
  
  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    set -e -x -u
    export DEBIAN_FRONTEND=noninteractive

    #change the source.list
    sudo apt-get update
    sudo apt-get install -y vim git cmake build-essential tcpdump tig jq tmux open-iscsi nfs-common wget software-properties-common 
    # Install ntp
    sudo apt-get install -y ntp
    # Install necessary packages for containerd
    sudo apt-get install -y ca-certificates curl gnupg2 lsb-release apt-transport-https 
	  # add containerd's official GPG key
	  sudo mkdir -p /etc/apt/keyrings
	  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
	  #setup Containerd repository
	  echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list 
    # Install Containerd 
	  sudo apt-get update
	  sudo apt-get install -y containerd.io
	
    # Install Kubernetes
    export KUBE_VERSION="1.24.10"
  	sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
	  echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo apt-get update
    sudo apt-get install -y kubeadm=${KUBE_VERSION}-00 kubelet=${KUBE_VERSION}-00 kubectl=${KUBE_VERSION}-00 
	  sudo apt-mark hold kubelet kubeadm kubectl 
    
    # Disable swap
    sudo swapoff -a && sudo sysctl -w vm.swappiness=0
    sudo sed '/swap.img/d' -i /etc/fstab
    
    # Ensure Kubelet is running
    sudo systemctl enable --now kubelet
    
    #Ensure Kernel has modules
    sudo modprobe overlay
    sudo modprobe br_netfilter

    #Update netowrking to allow traffic
    echo "net.bridge.bridge-nf-call-ip6tables = 1" | sudo tee /etc/sysctl.d/kubernetes.conf
    echo "net.bridge.bridge-nf-call-iptables = 1"  | sudo tee -- append /etc/sysctl.d/kubernetes.conf
    echo "net.ipv4.ip_forward = 1" | sudo tee -- append /etc/sysctl.d/kubernetes.conf 
      
    sudo sysctl --system
    
	  # Configure containerd settings
    echo "overlay" | sudo tee /etc/modules-load.d/containerd.conf
    echo "br_netfilter" | sudo tee -- append /etc/modules-load.d/containerd.conf

    sudo sysctl --system

    # Configure containerd and restart
    sudo mkdir -p /etc/containerd
    containerd config default | sudo tee /etc/containerd/config.toml
    sudo systemctl restart containerd
    sudo systemctl enable containerd
	
	  # Install and configure crictl
    export VER="v1.24.0"
    wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VER/crictl-$VER-linux-amd64.tar.gz
    tar zxvf crictl-$VER-linux-amd64.tar.gz
    sudo mv crictl /usr/local/bin
	
	  # Set the endpoints to avoid the deprecation error
    sudo crictl config --set runtime-endpoint=unix:///run/containerd/containerd.sock --set image-endpoint=unix:///run/containerd/containerd.sock
	
	  # Add Helm tool
    wget https://get.helm.sh/helm-v3.9.0-linux-amd64.tar.gz
    tar -xf helm-v3.9.0-linux-amd64.tar.gz
    sudo cp linux-amd64/helm /usr/local/bin/

    #sudo kubeadm init --kubernetes-version v${KUBE_VERSION} --apiserver-advertise-address=172.17.8.101 --pod-network-cidr=10.244.0.0/16
    #mkdir -p $HOME/.kube
    #sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    #sudo chown $(id -u):$(id -g) $HOME/.kube/config
    
    # Use Calico as the network plugin
    #kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
    
  SHELL
  
  config.vm.provider :vmware_desktop do |v|
      v.vmx["numvcpus"] = "2"
      v.vmx[ "memsize"] = "2048"
  end
end
