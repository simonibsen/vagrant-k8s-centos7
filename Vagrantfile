$masterscript = <<-SCRIPT
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
swapoff -a
cat > /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes] 
name=Kubernetes 
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64 
enabled=1 
gpgcheck=1 
repo_gpgcheck=1 
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg 
EOF
yum install kubectl kubeadm kubelet docker lsof nc nmap -y
systemctl restart docker && systemctl enable docker
systemctl  restart kubelet && systemctl enable kubelet
echo "nohup sh -c 'while true; do cat /root/k8s-joiner.txt | nc -l  5000; done' &" >> /etc/rc.d/rc.local
echo "swapoff -a" >> /etc/rc.d/rc.local
modprobe br_netfilter
echo "echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables" >> /etc/rc.d/rc.local
chmod 755 /etc/rc.d/rc.local
echo "1" > /proc/sys/net/bridge/bridge-nf-call-iptables
echo "Doing all kinds of k8s stuff - it may take a while but do be patient..."
kubeadm init  --pod-network-cidr=10.244.0.0/16 --apiserver-cert-extra-sans 10.0.2.15  --apiserver-advertise-address 10.0.100.11 | grep "kubeadm join" > /root/k8s-joiner.txt
nohup sh -c 'while true; do cat ~/k8s-joiner.txt | nc -l  5000; done' &
mkdir -p /root/.kube
cp /etc/kubernetes/admin.conf /root/.kube/config
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml
SCRIPT

$nodescript = <<-SCRIPT
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
swapoff -a
cat > /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes 
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64 
enabled=1 
gpgcheck=1 
repo_gpgcheck=1 
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg 
EOF
yum  install kubectl kubeadm kubelet docker lsof -y
echo "swapoff -a" >> /etc/rc.d/rc.local
echo "echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables" >> /etc/rc.d/rc.local
chmod 755 /etc/rc.d/rc.local
echo "Requires=rc-local.service" >> /etc/systemd/system/kubelet.service
systemctl restart docker && systemctl enable docker
modprobe br_netfilter
echo "1" > /proc/sys/net/bridge/bridge-nf-call-iptables
exec `curl 10.0.100.11:5000`
SCRIPT

Vagrant.configure("2") do |config|

    config.vm.define :k8s0 do |k8s0_config|
        k8s0_config.vm.box = "centos/7"
        k8s0_config.vm.hostname = "k8s0"
        k8s0_config.vm.network :private_network, ip: "10.0.100.11"
        k8s0_config.vm.provider "virtualbox" do |p|
            p.memory = "1024"
        end
        k8s0_config.vm.provision "shell", inline: $masterscript
    end

    config.vm.define :k8s1 do |k8s1_config|
        k8s1_config.vm.box = "centos/7"
        k8s1_config.vm.hostname = "k8s1"
        k8s1_config.vm.network :private_network, ip: "10.0.100.12"
        k8s1_config.vm.provider "virtualbox" do |p|
            p.memory = "1024"
        end
        k8s1_config.vm.provision "shell", inline: $nodescript
    end

    config.vm.define :k8s2 do |k8s2_config|
        k8s2_config.vm.box = "centos/7"
        k8s2_config.vm.hostname = "k8s2"
        k8s2_config.vm.network :private_network, ip: "10.0.100.13"
        k8s2_config.vm.provider "virtualbox" do |p|
            p.memory = "1024"
        end
        k8s2_config.vm.provision "shell", inline: $nodescript
    end

end
