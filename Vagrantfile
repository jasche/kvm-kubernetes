# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'

# Check required plugins
REQUIRED_PLUGINS_LIBVIRT = %w(vagrant-libvirt)
exit unless REQUIRED_PLUGINS_LIBVIRT.all? do |plugin|
  Vagrant.has_plugin?(plugin) || (
    puts "The #{plugin} plugin is required. Please install it with:"
    puts "$ vagrant plugin install #{plugin}"
    false
  )
end

$script = <<-SCRIPT
echo I am provisioning...
date > /etc/vagrant_provisioned_at

swapoff -a
sed -i '/.*swap/s/^/#/' /etc/fstab

yum update

yum install docker -y

systemctl enable docker
systemctl start docker

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF

# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable kubelet && systemctl start kubelet

cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

SCRIPT

Vagrant.configure("2") do |config|
  config.vm.define "master" do |master|
     master.vm.provider "libvirt" do |v|
        v.memory = 1024
        v.cpus = 2
     end
     master.vm.box = "centos/7"
     master.vm.hostname = "master"
     master.vm.network :private_network, ip: "192.168.34.11"
     master.vm.provision "shell", inline: $script
     master.vm.provision "bootstrap", type: "shell" do |s1|
    	s1.inline = $script
     end
     master.vm.provision "bootstrap", type: "shell" do |s2|
        s2.inline = "kubeadm init --pod-network-cidr=10.244.0.0/16"
     end

  end

  config.vm.define "node1" do |node1|
     node1.vm.box = "centos/7"
     node1.vm.hostname = "node1"
     node1.vm.network :private_network, ip: "192.168.34.12"
     node1.vm.provision "shell", inline: $script
  end

  config.vm.define "node2" do |node1|
     node1.vm.box = "centos/7"
     node1.vm.hostname = "node2"
     node1.vm.network :private_network, ip: "192.168.34.13"
     node1.vm.provision "shell", inline: $script
  end

end
