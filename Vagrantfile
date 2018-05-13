## Command used to generate Vagrantfile:['/Users/simon/bin/vfg', '-n', 'k8s', '-c3']
$script = <<-SCRIPT
exec bash
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
echo [kubernetes] > /etc/yum.repos.d/kubernetes.repo
echo name=Kubernetes >> /etc/yum.repos.d/kubernetes.repo
echo baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64 >> /etc/yum.repos.d/kubernetes.repo
echo enabled=1 >> /etc/yum.repos.d/kubernetes.repo
echo gpgcheck=1 >> /etc/yum.repos.d/kubernetes.repo
echo repo_gpgcheck=1 >> /etc/yum.repos.d/kubernetes.repo
echo gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg >> /etc/yum.repos.d/kubernetes.repo
yum install kubeadm docker -y
systemctl restart docker && systemctl enable docker
systemctl  restart kubelet && systemctl enable kubelet
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
sed -i '9s/^/Environment="KUBELET_EXTRA_ARGS=--fail-swap-on=false"\n/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
#swapoff -a
kubeadm init
mkdir -p \$HOME/.kube
cp /etc/kubernetes/admin.conf \$HOME/.kube/config
export kubever=\$(kubectl version | base64 | tr -d '\n')
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=\$kubever"
SCRIPT

Vagrant.configure("2") do |config|

    config.vm.define :k8s0 do |k8s0_config|
        k8s0_config.vm.box = "centos/7"
        k8s0_config.vm.hostname = "k8s0"
        k8s0_config.vm.network :private_network, ip: "10.0.100.11"
        k8s0_config.vm.provider "virtualbox" do |p|
            p.memory = "512"
        end
        k8s0_config.vm.provision "shell", inline: $script
    end

    config.vm.define :k8s1 do |k8s1_config|
        k8s1_config.vm.box = "centos/7"
        k8s1_config.vm.hostname = "k8s1"
        k8s1_config.vm.network :private_network, ip: "10.0.100.12"
        k8s1_config.vm.provider "virtualbox" do |p|
            p.memory = "256"
        end
    end

    config.vm.define :k8s2 do |k8s2_config|
        k8s2_config.vm.box = "centos/7"
        k8s2_config.vm.hostname = "k8s2"
        k8s2_config.vm.network :private_network, ip: "10.0.100.13"
        k8s2_config.vm.provider "virtualbox" do |p|
            p.memory = "256"
        end
    end

end
