# K3S Multi Master Cluster Vagrant configuration
Designed to istantiate a multi-master k3s 1.0.x cluster via Vagrant and VirtualBox.
It containes 4 machines
* master1
* master2
* node1
* node2

It is based on the work of Michaelc0n available here
http://devnetstack.com/kubernetes-the-easy-way-with-k3s/

And is taking inspiration also from the work of Osvaldo Toja available here
https://medium.com/@toja/running-k3s-with-metallb-on-vagrant-bd9603a5113b

# Functionality
The main functionality of multi-master is currently broken. The generic vagrant configuration created by Michaelc0n is working, while the second master is not.
