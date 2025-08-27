Vagrant.configure("2") do |config|
  # Box base
  BOX_NAME = "ubuntu/focal64"

  # Configurações comuns para todas as VMs
  config.vm.box = BOX_NAME
  config.vm.provider "virtualbox" do |vb|
    vb.memory = 1024
    vb.cpus = 1
  end

  # Script para instalar Docker
  INSTALL_DOCKER = <<-SHELL
    apt-get update -y
    apt-get install -y apt-transport-https ca-certificates curl software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
    apt-get update -y
    apt-get install -y docker-ce
    usermod -aG docker vagrant
  SHELL

  # Master (manager)
  config.vm.define "master" do |master|
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "192.168.56.10"
    master.vm.provision "shell", inline: INSTALL_DOCKER

    # Inicializa o swarm
    master.vm.provision "shell", inline: <<-SHELL
      docker swarm init --advertise-addr 192.168.56.10
      docker swarm join-token worker -q > /vagrant/worker_token.txt
    SHELL
  end

  # Nodes (workers)
  ["node01", "node02", "node03"].each_with_index do |name, i|
    config.vm.define name do |node|
      node.vm.hostname = name
      node.vm.network "private_network", ip: "192.168.56.#{11+i}"
      node.vm.provision "shell", inline: INSTALL_DOCKER

      # Adiciona ao cluster usando o token salvo
      node.vm.provision "shell", inline: <<-SHELL
        SWARM_TOKEN=$(cat /vagrant/worker_token.txt)
        docker swarm join --token $SWARM_TOKEN 192.168.56.10:2377
      SHELL
    end
  end
end