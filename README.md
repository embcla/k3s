
                                                                                             
               ██╗  ██╗██████╗ ███████╗██████╗  ██████╗ ██╗     ██╗                          
               ██║ ██╔╝╚════██╗██╔════╝██╔══██╗██╔═══██╗██║     ██║                          
               █████╔╝  █████╔╝███████╗██████╔╝██║   ██║██║     ██║                          
               ██╔═██╗  ╚═══██╗╚════██║██╔══██╗██║   ██║██║     ██║                          
               ██║  ██╗██████╔╝███████║██║  ██║╚██████╔╝███████╗███████╗                     
               ╚═╝  ╚═╝╚═════╝ ╚══════╝╚═╝  ╚═╝ ╚═════╝ ╚══════╝╚══════╝                     
                                                                                             
         A complete cluster in a small configurable space, like your laptop!                 

With immesurable thanks to:                                                              
*  My wife, whose patience is higher than my nerdiness                                  
*  Google, for developing and releasing K8S for free                                   
*  Rancher Labs, for developing and releasing K3S for free                             
*  Osvaldo Toja, for publishing his inspiring work on Github and Medium https://medium.com/@toja/running-k3s-with-metallb-on-vagrant-bd9603a5113b        
*  Michaelc0n, for publishing his work on Github http://devnetstack.com/kubernetes-the-easy-way-with-k3s/

# 1) General description of the project                                                    
Kubernetes is a very complex environment: it disentangles and virtualizes every       
resource, giving the administrator immense power. In the best of traditions           
with immense power come immense responsibilities, specifically to learn about         
all the new reources to configure in order to obtain the expected result.             
Testing anything like this used to be a costly undertaking involving Google Cloud     
Engine, Digital Ocean, Amazon Kubernetes Engine, Azure, etc. With K3S you can         
run a 90% Kubernetes environment right in your computer.                              


# 2) Prerequisites                                                                         
*  Virtualbox (a version compatible with vagrant)                                     
*  Vagrant                                                                            
*  working internet connection (to download packages for Linux and K3S)               
*  a "persistentvolume" folder in the current path, it get mounted in the nodes       


# 3) Usage                                                                                 
Run "vagrant up" and it will create all the cluster for you.                          
If vagrant gets stuck during spinning up of the K3S service on the masters,           
give it a ctrl+c and relunch it. This is a known bug.                                 

Once creation, booting and provisioning of all machines is completed, you can         
log into master1 and type "k3s kubectl get nodes,pods,svc -A" and you'll see          
your cluster. Happy clustering!                                                       


# 4) Configuration                                                                         
You can configure a number of options in this Vagrantfile, namely                     
master nodes, worker nodes, memory and cpu of these two types, cluster name prefix    

# 5) Kubectl and helm
Kubectl and helm both work out of the box, the configuration of the cluster is saved as k3s.yaml.
You can run kubectl --kubeconfig k3s.yaml get nodes to see it working
An easy way to get around the verbose notation is aliasing the command
alias k="kubectl --kubeconfig k3s.yaml"
and after this you can just type "k get nodes"

# 6) Known bugs
* K3S install script gets stuck during master nodes installation. I haven't been able to figure out why yet.
                                                                                             


# To Dos
Fix mysql single point of failure<br/>
Run something useful<br/>
