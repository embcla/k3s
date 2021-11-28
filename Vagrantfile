###############################################################################################
#                                                                                             #
#               ██╗  ██╗██████╗ ███████╗██████╗  ██████╗ ██╗     ██╗                          #
#               ██║ ██╔╝╚════██╗██╔════╝██╔══██╗██╔═══██╗██║     ██║                          #
#               █████╔╝  █████╔╝███████╗██████╔╝██║   ██║██║     ██║                          #
#               ██╔═██╗  ╚═══██╗╚════██║██╔══██╗██║   ██║██║     ██║                          #
#               ██║  ██╗██████╔╝███████║██║  ██║╚██████╔╝███████╗███████╗                     #
#               ╚═╝  ╚═╝╚═════╝ ╚══════╝╚═╝  ╚═╝ ╚═════╝ ╚══════╝╚══════╝                     #
#                                                                                             #
#         A complete cluster in a small configurable space, like your laptop!                 #
#                                                                                             #
#    With immesurable thanks to:                                                              #
#      °  My wife, whos patience is higher than my nerdiness                                  #
#      *  Google, for developing and releasing K8S for free                                   #
#      *  Rancher Labs, for developing and releasing K3S for free                             #
#      *  Osvaldo Toja, for publishing his inspiring work on Github and Medium                #
#            https://medium.com/@toja/running-k3s-with-metallb-on-vagrant-bd9603a5113b        #
#                                                                                             #
#                                                                                             #
#    1) General description of the project                                                    #
#       Kubernetes is a very complex environment: it disentangles and virtualizes every       #
#       resource, giving the administrator immense power. In the best of traditions           #
#       with immense power come immense responsibilities, specifically to learn about         #
#       all the new reources to configure in order to obtain the expected result.             #
#       Testing anything like this used to be a costly undertaking involving Google Cloud     #
#       Engine, Digital Ocean, Amazon Kubernetes Engine, Azure, etc. With K3S you can         #
#       run a 90% Kubernetes environment right in your computer.                              #
#                                                                                             #
#                                                                                             #
#    2) Prerequisites                                                                         #
#       *  Virtualbox (a version compatible with vagrant)                                     #
#       *  Vagrant                                                                            #
#       *  working internet connection (to download packages for Linux and K3S)               #
#       *  a "persistentvolume" folder in the current path, it get mounted in the nodes       #
#                                                                                             #
#                                                                                             #
#    3) Usage                                                                                 #
#       Run "vagrant up" and it will create all the cluster for you.                          #
#       If vagrant gets stuck during spinning up of the K3S service on the masters,           #
#       give it a ctrl+c and relunch it. This is a known bug.                                 #
#                                                                                             #
#       Once creation, booting and provisioning of all machines is completed, you can         #
#       log into master1 and type "k3s kubectl get nodes,pods,svc -A" and you'll see          #
#       your cluster. Happy clustering!                                                       #
#                                                                                             #
#                                                                                             #
#    4) Configuration                                                                         #
#       You can configure a number of options in this Vagrantfile, namely                     #
#       master nodes, worker nodes, memory and cpu of these two types, cluster name prefix    #
#                                                                                             #
#                                                                                             #
#                                                                                             #
#                                                                                             #
#                                                                                             #
#                                                                                             #
###############################################################################################


default_box = 'ubuntu/bionic64'

# this is a prefix to the name of each node
cluster_name = 'ha'

# Number of worker nodes to be created, numbered progressively from 1
cluster_conf_workernode_num = 3

# Number of master nodes to be created, numbered progressively from 1
#    NODE: master nodes are tainted, they do not execute
cluster_conf_masternode_num = 2

# Memory allocation for master type VM and worker node type VM
cluster_conf_memory_alloc_master = 1250
cluster_conf_memory_alloc_node = 850

# CPU allocation for master type VM and worker node type VM
cluster_conf_cpu_alloc_master = 1
cluster_conf_cpu_alloc_node = 1

$configureK3SClusterInit=<<-SHELL
  NODE_TOKEN=$(echo $(hostname) $(date +%s) | shasum | base64)
  echo ${NODE_TOKEN} > /vagrant/node-token
SHELL

$configureMySQLServer=<<-SHELL
  echo "Provisioning MySQL database and user for k3s"
  sudo apt install mysql-server mysql-client -y
  MYSQL_PASSWORD=$(echo $(hostname) $(date +%s) | shasum | base64)
  sudo mysql -e "drop user k3s;"
  sudo mysql -e "drop database k3s;"
  sudo mysql -e "create database k3s;"
  sudo mysql -e "grant all privileges on k3s.* to k3s@'%' identified by '${MYSQL_PASSWORD}';"
  sudo mysql -e "flush privileges;"
  echo ${MYSQL_PASSWORD} > /vagrant/mysql_password
  sudo sed -i s/bind/#bind/g /etc/mysql/mysql.conf.d/mysqld.cnf
  sudo systemctl restart mysql
  IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)
  echo $IPADDR > /vagrant/mysql_ip
SHELL

$configureK3SMaster=<<-SHELL
  NODE_TOKEN=$(cat /vagrant/node-token)
  MYSQL_IP=$(cat /vagrant/mysql_ip)
  MYSQL_PASSWORD=$(cat /vagrant/mysql_password)
  HOSTNAME=$(hostname)
  IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)
  #curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC=" -v 2 -l /vagrant/master1.log --node-ip=${IPADDR} --flannel-iface=enp0s8 --write-kubeconfig-mode 644 --kube-apiserver-arg="service-node-port-range=30000-30100" --no-deploy=servicelb --no-deploy=traefik" K3S_DATASTORE_ENDPOINT="http://127.0.0.1:2379" sh -
  curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="-v 2 -l /vagrant/logs/${HOSTNAME}_node.log -t ${NODE_TOKEN} --flannel-iface=enp0s8 --write-kubeconfig-mode 644 --tls-san k3s-cluster-01.lan --no-deploy servicelb --no-deploy traefik --node-taint k3s-controlplane=true:NoExecute --no-deploy=traefik --no-deploy=servicelb --datastore-endpoint mysql://k3s:${MYSQL_PASSWORD}@tcp(${MYSQL_IP}:3306)/k3s" sh -
  #hostnamectl set-hostname master1
  #NODE_TOKEN="/var/lib/rancher/k3s/server/node-token"
  sudo sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
  sudo systemctl reload ssh
  KUBE_CONFIG="/etc/rancher/k3s/k3s.yaml"
SHELL

$configureK3SNode=<<-SHELL
  NODE_TOKEN=$(cat /vagrant/node-token)
  MYSQL_IP=$(cat /vagrant/mysql_ip)
  IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)
  curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent -v 2 -l /vagrant/logs/${HOSTNAME}_node.log -t ${NODE_TOKEN} --node-ip=${IPADDR} --flannel-iface=enp0s8 --server https://${MYSQL_IP}:6443" sh -
  sudo sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
  sudo systemctl reload ssh
SHELL

Vagrant.configure(2) do |config|

  (1..cluster_conf_workernode_num+cluster_conf_masternode_num).each do |i|
    if i <= cluster_conf_masternode_num then
      num_string = "#{i}"
      vm_name="#{cluster_name}master#{num_string}"
    else
      num_string = "#{i-cluster_conf_masternode_num}"
      vm_name="#{cluster_name}node#{num_string}"
    end
    config.vm.graceful_halt_timeout = 150
    config.vm.define vm_name do |s|
      s.vm.box = default_box
      s.vm.hostname = vm_name
      s.vm.synced_folder ".", "/vagrant", type:"virtualbox"
      s.vm.synced_folder "./persistentstorage", "/persistentstorage", type:"virtualbox"
      if i == 1 then
        s.vm.network "forwarded_port", guest: 6443, host: 6443 # ACCESS K8S API
      end
      if i <= cluster_conf_masternode_num then
        private_ip = "192.168.0.#{250+i}"
        host_port_1 = "225#{i}"
        host_port_2 = "205#{i}"
      else
        private_ip = "192.168.0.#{200+i}"
        host_port_1 = "220#{i}"
        host_port_2 = "200#{i}"
      end
      s.vm.network 'private_network', ip: private_ip,  virtualbox__intnet: false
      s.vm.network "forwarded_port", guest: 22, host: host_port_1, id: "ssh", disabled: true
      s.vm.network "forwarded_port", guest: 22, host: host_port_2 # SSH TO MASTER/NODE
      #for p in 30000..30100 # PORTS DEFINED FOR K8S TYPE-NODE-PORT ACCESS
      #  s.vm.network "forwarded_port", guest: p, host: p, protocol: "tcp"
      #end
      s.vm.provider "virtualbox" do |v|
        if i <= cluster_conf_masternode_num then
          v.memory = cluster_conf_memory_alloc_master
          v.cpus = cluster_conf_cpu_alloc_master
        else
          v.memory = cluster_conf_memory_alloc_node
          v.cpus = cluster_conf_cpu_alloc_node
        end
        v.name = vm_name

      end
      s.vm.hostname = vm_name
      if i == 1 then
        s.vm.provision "shell", privileged: false, inline: <<-SHELL
          echo "Generating cluster security token"
        SHELL
        s.vm.provision "shell", inline: $configureK3SClusterInit
      end

      s.vm.provision "shell", privileged: false, inline: <<-SHELL
        echo "Setting hostname inside the VM to #{vm_name}"
        echo "Local server address is http://#{$hostname}"
      SHELL

      s.vm.provision "shell", privileged: false, inline: <<-SHELL
        echo "Applying configuration, step 1: common configuration"
      SHELL
      if i <= cluster_conf_masternode_num then
        s.vm.provision "shell", privileged: false, inline: <<-SHELL
          echo "Applying configuration, step 2: master node configuration"
        SHELL
        if i == 1 then
          s.vm.provision "shell", inline: $configureMySQLServer
        end
        s.vm.provision "shell", inline: $configureK3SMaster
      else
        s.vm.provision "shell", privileged: false, inline: <<-SHELL
          echo "Applying configuration, step 2: worker node configuration"
        SHELL
        s.vm.provision "shell", inline: $configureK3SNode
      end

      if i == 1 then
        s.vm.provision "shell", inline: <<-SHELL
          KUBE_CONFIG="/etc/rancher/k3s/k3s.yaml"
          cp ${KUBE_CONFIG} /vagrant/
        SHELL
      end
    end
  end
end
