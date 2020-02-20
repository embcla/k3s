# K3S Multi Master Cluster Vagrant configuration
Designed to istantiate a multi-master k3s 1.0.x cluster via Vagrant and VirtualBox.
It containes 4 machines
* master1
* master2
* node1
* node2

It is based on the work of Michaelc0n available here<br/>
http://devnetstack.com/kubernetes-the-easy-way-with-k3s/

And is taking inspiration also from the work of Osvaldo Toja available here<br/>
https://medium.com/@toja/running-k3s-with-metallb-on-vagrant-bd9603a5113b

# Functionality
Moved to a configuration similar to that of the Rancher K3S Masterclass of 17th December 2019 ( https://www.youtube.com/watch?v=_lNnrrp-8zQ ): external MySQL database, external token secret. All machines load the same token, masters use the MySQL for datastore.

This way the cluster does come up showing two masters, but tainting the masters doesn't work: master1 still executes all pods.

# NodeRed
Added a JSON flow definition to extract data for each room from Tado, parse it, clean it and retransmit it over MQTT.
MQTT stream will then be routed through prometheus and grafana for analysis and graphing
![NodeRed Tado Flow](/Tado%20NodeRed%20Flow.PNG)

# To Do
Fix tainting<br/>
Fix virtual ip<br/>
Fix mysql single point of failure<br/>
Run something useful<br/>
