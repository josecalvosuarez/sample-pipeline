# Vagrantfile for setting up 4 VMs on VirtualBox
# Supports: ARM64 Mac (Apple Silicon), x86_64 Mac (Intel), and Windows x86_64
require 'rbconfig'

Vagrant.configure("2") do |config|
  # Detect host CPU architecture to select the correct Vagrant box and binaries
  host_cpu = RbConfig::CONFIG['host_cpu']
  is_arm = host_cpu.match?(/arm|aarch/i)

  box_name      = is_arm ? "net9/ubuntu-24.04-arm64" : "bento/ubuntu-24.04"
  minikube_arch = is_arm ? "arm64" : "amd64"

  # Define a private network for communication
  network = "192.168.56."

  # Shared configurations for all VMs
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
    vb.gui = false
  end

  # Jenkins VM
  config.vm.define "jenkins" do |jenkins|
    jenkins.vm.box = box_name
    jenkins.vm.hostname = "jenkins.local"
    jenkins.vm.network "private_network", ip: "#{network}101"
    jenkins.hostmanager.aliases = ["jenkins.local"]
    jenkins.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y openjdk-21-jdk docker.io
      sudo systemctl enable docker
      sudo systemctl start docker
      sudo usermod -aG docker jenkins
      curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
      echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
      sudo apt-get update
      sudo apt-get install -y jenkins
      sudo systemctl enable jenkins
      sudo systemctl start jenkins
    SHELL
  end

  # Git Repository VM
  config.vm.define "git" do |git|
    git.vm.box = box_name
    git.vm.hostname = "git.local"
    git.vm.network "private_network", ip: "#{network}102"
    git.hostmanager.aliases = ["git.local"]
    git.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y git
    SHELL
  end

  # Configuration Management VM
  config.vm.define "config-mgmt" do |config_mgmt|
    config_mgmt.vm.box = box_name
    config_mgmt.vm.hostname = "config-mgmt.local"
    config_mgmt.vm.network "private_network", ip: "#{network}103"
    config_mgmt.hostmanager.aliases = ["config-mgmt.local"]
    config_mgmt.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y software-properties-common
      sudo add-apt-repository --yes --update ppa:ansible/ansible
      sudo apt-get install -y ansible
    SHELL
  end

  # Minikube VM
  config.vm.define "minikube" do |minikube|
    minikube.vm.box = box_name
    minikube.vm.hostname = "minikube.local"
    minikube.vm.network "private_network", ip: "#{network}104"
    minikube.hostmanager.aliases = ["minikube.local"]
    minikube.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
      vb.cpus = 2
    end
    minikube.vm.provision "shell", env: { "MINIKUBE_ARCH" => minikube_arch }, inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y curl apt-transport-https docker.io
      sudo systemctl enable docker
      sudo systemctl start docker
      sudo usermod -aG docker vagrant

      curl -LO "https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-${MINIKUBE_ARCH}"
      sudo install "minikube-linux-${MINIKUBE_ARCH}" /usr/local/bin/minikube
      rm "minikube-linux-${MINIKUBE_ARCH}"

      KUBECTL_VERSION=$(curl -L -s https://dl.k8s.io/release/stable.txt)
      curl -LO "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${MINIKUBE_ARCH}/kubectl"
      chmod +x kubectl
      sudo mv ./kubectl /usr/local/bin/kubectl

      sudo -u vagrant minikube config set driver docker
    SHELL
  end

  # Enable hostmanager plugin
  # NOTE: On Windows, run your terminal as Administrator so hostmanager can
  # write to C:\Windows\System32\drivers\etc\hosts
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.ignore_private_ip = false
end
