default_box = 'ubuntu/bionic64'

Vagrant.configure(2) do |config|
  config.vm.define 'master1' do |master1|
    master1.vm.box = default_box
    master1.vm.hostname = "master1"
    master1.vm.synced_folder ".", "/vagrant", type:"virtualbox"
    master1.vm.network 'private_network', ip: "192.168.0.200",  virtualbox__intnet: true
    master1.vm.network "forwarded_port", guest: 22, host: 2222, id: "ssh", disabled: true
    master1.vm.network "forwarded_port", guest: 22, host: 2000 # SSH TO MASTER/NODE
    master1.vm.network "forwarded_port", guest: 6443, host: 6443 # ACCESS K8S API
    for p in 30000..30100 # PORTS DEFINED FOR K8S TYPE-NODE-PORT ACCESS
      master1.vm.network "forwarded_port", guest: p, host: p, protocol: "tcp"
      end
    master1.vm.provider "virtualbox" do |v|
      v.memory = "850"
      v.name = "master1"
      v.cpus = 1
      end

    master1.vm.provision "shell", inline: <<-SHELL
      sudo apt update
      sudo apt install etcd-server etcd-client -y
      IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)
      curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC=" -v 2 -l /vagrant/master1.log --node-ip=${IPADDR} --flannel-iface=enp0s8 --write-kubeconfig-mode 644 --kube-apiserver-arg="service-node-port-range=30000-30100" --no-deploy=servicelb --no-deploy=traefik" K3S_DATASTORE_ENDPOINT="http://127.0.0.1:2379" sh -
      hostnamectl set-hostname master1
      NODE_TOKEN="/var/lib/rancher/k3s/server/node-token"
      while [ ! -e ${NODE_TOKEN} ]
      do
          sleep 1
      done
      cat ${NODE_TOKEN}
      cp ${NODE_TOKEN} /vagrant/
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
    master2.vm.network 'private_network', ip: "192.168.0.201",  virtualbox__intnet: true
    master2.vm.network "forwarded_port", guest: 22, host: 2222, id: "ssh", disabled: true
    master2.vm.network "forwarded_port", guest: 22, host: 2001 # SSH TO MASTER/NODE
    master2.vm.network "forwarded_port", guest: 6443, host: 6444 # ACCESS K8S API
    for p in 30000..30100 # PORTS DEFINED FOR K8S TYPE-NODE-PORT ACCESS
      master2.vm.network "forwarded_port", guest: p, host: p, protocol: "tcp"
      end
    master2.vm.provider "virtualbox" do |v|
      v.memory = "850"
      v.name = "master2"
      v.cpus = 1
      end

    master2.vm.provision "shell", inline: <<-SHELL
      sudo apt update
      sudo apt install etcd-server etcd-client -y
      IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)
      curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server -v 2 -l /vagrant/master2.log --node-ip=${IPADDR} --server https://192.168.0.200:6343 --flannel-iface=enp0s8 --write-kubeconfig-mode 644 --kube-apiserver-arg="service-node-port-range=30000-30100" --no-deploy=servicelb --no-deploy=traefik" K3S_DATASTORE_ENDPOINT="https://192.168.0.200:2379" K3S_TOKEN=$(cat /vagrant/node-token) sh -
      hostnamectl set-hostname master2
      NODE_TOKEN="/var/lib/rancher/k3s/server/node-token"
      while [ ! -e ${NODE_TOKEN} ]
      do
          sleep 1
      done
      cat ${NODE_TOKEN}
      cp ${NODE_TOKEN} /vagrant/
      sudo sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
      sudo systemctl reload ssh
      KUBE_CONFIG="/etc/rancher/k3s/k3s.yaml"
      cp ${KUBE_CONFIG} /vagrant/ #copy contents of "k3s.yaml" to ".kube/config" to 'kubectl' from local-machine
    SHELL
  end
  
  config.vm.define 'node1' do |node1|
    node1.vm.box = default_box
    node1.vm.hostname = "node1"
    node1.vm.synced_folder ".", "/vagrant", type:"virtualbox"
    node1.vm.network 'private_network', ip: "192.168.0.202",  virtualbox__intnet: true
    node1.vm.network "forwarded_port", guest: 22, host: 2222, id: "ssh", disabled: true
    node1.vm.network "forwarded_port", guest: 22, host: 2002
    node1.vm.provider "virtualbox" do |v|
      v.memory = "850"
      v.name = "node1"
      v.cpus = 1
      end
    
    node1.vm.provision "shell", inline: <<-SHELL
      IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)
      curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC=" -v 2 -l /vagrant/node1.log  --node-ip=${IPADDR} --flannel-iface=enp0s8" K3S_URL=https://192.168.0.200:6443 K3S_TOKEN=$(cat /vagrant/node-token) sh -
      sudo sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
      sudo systemctl reload ssh
    SHELL
  end

  config.vm.define 'node2' do |node2|
    node2.vm.box = default_box
    node2.vm.hostname = "node2"
    node2.vm.synced_folder ".", "/vagrant", type:"virtualbox"
    node2.vm.network 'private_network', ip: "192.168.0.203",  virtualbox__intnet: true
    node2.vm.network "forwarded_port", guest: 22, host: 2222, id: "ssh", disabled: true
    node2.vm.network "forwarded_port", guest: 22, host: 2003
    node2.vm.provider "virtualbox" do |v|
      v.memory = "850"
      v.name = "node2"
      v.cpus = 1
      end
    
    node2.vm.provision "shell", inline: <<-SHELL
      IPADDR=$(ip a show enp0s8 | grep "inet " | awk '{print $2}' | cut -d / -f1)
      curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC=" -v 2 -l /vagrant/node2.log  --node-ip=${IPADDR} --flannel-iface=enp0s8" K3S_URL=https://192.168.0.200:6443 K3S_TOKEN=$(cat /vagrant/node-token) sh -
      rm -f /vagrant/node-token
      sudo sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
      sudo systemctl reload ssh
    SHELL
  end

end