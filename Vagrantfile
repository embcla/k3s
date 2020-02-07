default_box = 'ubuntu/bionic64'


Vagrant.configure(2) do |config|
  config.vm.graceful_halt_timeout = 150
  config.vm.define 'master1' do |master1|
    master1.vm.box = default_box
    master1.vm.hostname = "master1"
    master1.vm.synced_folder ".", "/vagrant", type:"virtualbox"
    master1.vm.network 'private_network', ip: "192.169.1.251",  virtualbox__intnet: false
    master1.vm.network "forwarded_port", guest: 22, host: 2231, id: "ssh", disabled: true
    master1.vm.network "forwarded_port", guest: 22, host: 2031 # SSH TO MASTER/NODE
    master1.vm.network "forwarded_port", guest: 6443, host: 6443 # ACCESS K8S API
    for p in 30000..30100 # PORTS DEFINED FOR K8S TYPE-NODE-PORT ACCESS
      master1.vm.network "forwarded_port", guest: p, host: p, protocol: "tcp"
      end
    master1.vm.provider "virtualbox" do |v|
      v.memory = "1250"
      v.name = "master1"
      v.cpus = 1
      end

    $mysql_setup_script = <<-SCRIPT
      echo "Provisioning MySQL database and user for k3s"
      MYSQL_PASSWORD=$(echo $(hostname) $(date +%s) | shasum | base64)
      sudo mysql -e "drop user k3s;"
      sudo mysql -e "drop database k3s;"
      sudo mysql -e "create database k3s;"
      sudo mysql -e "grant all privileges on k3s.* to k3s@'%' identified by '${MYSQL_PASSWORD}';"
      sudo mysql -e "flush privileges;"
      echo ${MYSQL_PASSWORD} > /vagrant/mysql_password
      sudo sed -i s/bind/#bind/g /etc/mysql/mysql.conf.d/mysqld.cnf
      sudo systemctl restart mysql 
    SCRIPT

    master1.vm.provision "shell", inline: <<-SHELL
      sudo apt update
      sudo apt install mysql-server mysql-client -y
    SHELL

    master1.vm.provision "shell", inline: $mysql_setup_script

    master1.vm.provision "shell", inline: <<-SHELL
      NODE_TOKEN=$(echo $(hostname) $(date +%s) | shasum | base64)
      echo ${NODE_TOKEN} > /vagrant/node-token
      MYSQL_PASSWORD=$(cat /vagrant/mysql_password)
      IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)
      #curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC=" -v 2 -l /vagrant/master1.log --node-ip=${IPADDR} --flannel-iface=enp0s8 --write-kubeconfig-mode 644 --kube-apiserver-arg="service-node-port-range=30000-30100" --no-deploy=servicelb --no-deploy=traefik" K3S_DATASTORE_ENDPOINT="http://127.0.0.1:2379" sh -
      curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="-v 2 -l /vagrant/master1.log -t ${NODE_TOKEN} --flannel-iface=enp0s8 --no-deploy traefik --no-deploy servicelb --write-kubeconfig-mode 644 --tls-san k3s-cluster-01.lan --node-taint k3s-controlplane=true:NoExecute --datastore-endpoint mysql://k3s:${MYSQL_PASSWORD}@tcp(${IPADDR}:3306)/k3s" sh -
      hostnamectl set-hostname master1
      #NODE_TOKEN="/var/lib/rancher/k3s/server/node-token"
      #while [ ! -e ${NODE_TOKEN} ]
      #do
      #    sleep 1
      #done
      #cat ${NODE_TOKEN}
      #cp ${NODE_TOKEN} /vagrant/
      sudo sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
      sudo systemctl reload ssh
      KUBE_CONFIG="/etc/rancher/k3s/k3s.yaml"
      cp ${KUBE_CONFIG} /vagrant/ #copy contents of "k3s.yaml" to ".kube/config" to 'kubectl' from local-machine
    SHELL
  end

  config.vm.define 'master2' do |master2|
    master2.vm.box = default_box
    master2.vm.hostname = "master2"
    master2.vm.synced_folder ".", "/vagrant", type:"virtualbox"
    master2.vm.network 'private_network', ip: "192.169.1.252",  virtualbox__intnet: false
    master2.vm.network "forwarded_port", guest: 22, host: 2232, id: "ssh", disabled: true
    master2.vm.network "forwarded_port", guest: 6443, host: 6444 # ACCESS K8S API
    master2.vm.network "forwarded_port", guest: 22, host: 2032 # SSH TO MASTER/NODE
    for p in 30000..30100 # PORTS DEFINED FOR K8S TYPE-NODE-PORT ACCESS
      master2.vm.network "forwarded_port", guest: p, host: p+101, protocol: "tcp"
      end
    master2.vm.provider "virtualbox" do |v|
      v.memory = "1250"
      v.name = "master2"
      v.cpus = 1
      end

    master2.vm.provision "shell", inline: <<-SHELL
      sudo apt update
      NODE_TOKEN=$(cat /vagrant/node-token)
      MYSQL_PASSWORD=$(cat /vagrant/mysql_password)
      IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)
      #curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server -v 2 -l /vagrant/master2.log --node-ip=${IPADDR} --server https://192.169.1.251:6343 --flannel-iface=enp0s8 --write-kubeconfig-mode 644 --kube-apiserver-arg="service-node-port-range=30000-30100" --no-deploy=servicelb --no-deploy=traefik" K3S_DATASTORE_ENDPOINT="https://192.169.1.251:2379" K3S_TOKEN=$(cat /vagrant/node-token) sh -
      curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="-v 2 -l /vagrant/master2.log -t ${NODE_TOKEN} --flannel-iface=enp0s8 --no-deploy traefik --no-deploy servicelb --write-kubeconfig-mode 644 --tls-san k3s-cluster-01.lan --node-taint k3s-controlplane=true:NoExecute --datastore-endpoint mysql://k3s:${MYSQL_PASSWORD}@tcp(192.169.1.251:3306)/k3s" sh -
      hostnamectl set-hostname master2
      sudo sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
      sudo systemctl reload ssh
      
    SHELL
  end
  
  config.vm.define 'node1' do |node1|
    node1.vm.box = default_box
    node1.vm.hostname = "node1"
    node1.vm.synced_folder ".", "/vagrant", type:"virtualbox"
    node1.vm.network 'private_network', ip: "192.169.1.201",  virtualbox__intnet: false
    node1.vm.network "forwarded_port", guest: 22, host: 2221, id: "ssh", disabled: true
    node1.vm.network "forwarded_port", guest: 22, host: 2001
    node1.vm.provider "virtualbox" do |v|
      v.memory = "850"
      v.name = "node1"
      v.cpus = 1
      end
    
    node1.vm.provision "shell", inline: <<-SHELL
      NODE_TOKEN=$(cat /vagrant/node-token)
      IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)
      curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent -v 2 -l /vagrant/node1.log -t ${NODE_TOKEN} --node-ip=${IPADDR} --flannel-iface=enp0s8 --server https://192.169.1.251:6443" sh -
      sudo sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
      sudo systemctl reload ssh
    SHELL
  end

  config.vm.define 'node2' do |node2|
    node2.vm.box = default_box
    node2.vm.hostname = "node2"
    node2.vm.synced_folder ".", "/vagrant", type:"virtualbox"
    node2.vm.network 'private_network', ip: "192.169.1.202",  virtualbox__intnet: false
    node2.vm.network "forwarded_port", guest: 22, host: 2222, id: "ssh", disabled: true
    node2.vm.network "forwarded_port", guest: 22, host: 2002
    node2.vm.provider "virtualbox" do |v|
      v.memory = "850"
      v.name = "node2"
      v.cpus = 1
      end
    
    node2.vm.provision "shell", inline: <<-SHELL
      NODE_TOKEN=$(cat /vagrant/node-token)
      IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)
      curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent -v 2 -l /vagrant/node2.log -t ${NODE_TOKEN} --node-ip=${IPADDR} --flannel-iface=enp0s8 --server https://192.169.1.251:6443" sh -
      sudo sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
      sudo systemctl reload ssh
    SHELL
  end
  
  config.vm.define 'node3' do |node3|
    node3.vm.box = default_box
    node3.vm.hostname = "node3"
    node3.vm.synced_folder ".", "/vagrant", type:"virtualbox"
    node3.vm.network 'private_network', ip: "192.169.1.203",  virtualbox__intnet: false
    node3.vm.network "forwarded_port", guest: 22, host: 2223, id: "ssh", disabled: true
    node3.vm.network "forwarded_port", guest: 22, host: 2003
    node3.vm.provider "virtualbox" do |v|
      v.memory = "850"
      v.name = "node3"
      v.cpus = 1
      end
    
    node3.vm.provision "shell", inline: <<-SHELL
      NODE_TOKEN=$(cat /vagrant/node-token)
      IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)
      curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent -v 2 -l /vagrant/node3.log -t ${NODE_TOKEN} --node-ip=${IPADDR} --flannel-iface=enp0s8 --server https://192.169.1.251:6443" sh -
      sudo sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
      sudo systemctl reload ssh
    SHELL
  end

  config.vm.define 'node4' do |node4|
    node4.vm.box = default_box
    node4.vm.hostname = "node4"
    node4.vm.synced_folder ".", "/vagrant", type:"virtualbox"
    node4.vm.network 'private_network', ip: "192.169.1.204",  virtualbox__intnet: false
    node4.vm.network "forwarded_port", guest: 22, host: 2224, id: "ssh", disabled: true
    node4.vm.network "forwarded_port", guest: 22, host: 2004
    node4.vm.provider "virtualbox" do |v|
      v.memory = "850"
      v.name = "node4"
      v.cpus = 1
      end
    
    node4.vm.provision "shell", inline: <<-SHELL
      NODE_TOKEN=$(cat /vagrant/node-token)
      IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)
      curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent -v 2 -l /vagrant/node4.log -t ${NODE_TOKEN} --node-ip=${IPADDR} --flannel-iface=enp0s8 --server https://192.169.1.251:6443" sh -
      sudo sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
      sudo systemctl reload ssh
    SHELL
  end

end