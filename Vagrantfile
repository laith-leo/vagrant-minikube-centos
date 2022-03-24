Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.define "minikube" do |kube|
  config.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
      vb.cpus = "4"
  end

  config.vm.network "forwarded_port",
      guest: 30000,
      host:  30000,
      auto_correct: true

  config.vm.network "forwarded_port",
      guest: 8001,
      host:  8001,
      auto_correct: true

  config.vm.network "forwarded_port",
      guest: 80,
      host:  80,
      auto_correct: true

  config.vm.network "forwarded_port",
      guest: 443,
      host:  443,
      auto_correct: true

    config.vm.provision "system_config", type: "shell",  inline: <<-SCRIPT
echo "Setting up system configurations.."
sudo bash -c "echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables"
sudo yum install yum-utils device-mapper-persistent-data lvm2 -y
sudo swapoff -a
sudo hostnamectl set-hostname minikube
SCRIPT

    config.vm.provision "docker", type: "shell",  inline: <<-SCRIPT
echo "Installing Docker and kubectl"
sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
[docker-ce-edge]
name=Docker CE Edge - $basearch
baseurl=https://download.docker.com/linux/centos/7/$basearch/edge
enabled=1
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg
EOF
sudo yum install -y docker-ce-18.06.1.ce-3.el7.x86_64
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -Gdocker vagrant
sudo systemctl daemon-reload && sudo systemctl restart docker
KUBE_VERSION=v1.23.1
sudo bash -c "curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/${KUBE_VERSION}/bin/linux/amd64/kubectl && chmod +x kubectl && mv -f kubectl /usr/bin/"
sudo echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
SCRIPT

    config.vm.provision "minikube", type: "shell",  inline: <<-SCRIPT
echo "Installing minikube"
sudo bash -c "curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && install minikube-linux-amd64 /usr/bin/minikube" && rm -rf minikube-linux-amd64
echo "Setting up and starting K8S"
minikube config set WantReportErrorPrompt false
sudo minikube start --vm-driver=none
SCRIPT

    config.vm.provision "shell", inline: <<-SHELL
#!/bin/bash
echo  "Configuring vagrant user directory"
printf "source <(kubectl completion bash)\n" >> /home/vagrant/.bashrc
# Permissions
sudo mkdir /home/vagrant/.kconfig
sudo cp /etc/kubernetes/admin.conf /home/vagrant/.kconfig
sudo chown vagrant:vagrant /home/vagrant/.kconfig/admin.conf
printf "export KUBECONFIG=/home/vagrant/.kconfig/admin.conf\n" >> /home/vagrant/.bashrc
source /home/vagrant/.bashrc
SHELL
 end

end



