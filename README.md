# vagrant-k8s-centos7
A quick and dirty Vagrant file to build a k8s cluster on Centos 7 VMs.  Edit the file to include additional nodes or change VM parameters as appropriate.  This assumes you have Vagrant and Virtualbox installed.  To build a k8s cluster type `vagrant up` and a 1 master, 2 node cluster will be built.

To use native k8s tools on your host system to interact with the underlying VMs do the following (these instructions assume you are using a Mac): 

1. In your Vagrant directory run: 
`vagrant ssh-config > vsshconfig`
This saves your Vagrant ssh config to a file for easier access later.  
2. Next do this to set up forwarding to be used in your k8s config (where k8s0 is the name of the k8s master):
`ssh -f -N -M -S ~/.ssh/vagrant.sock -L 16443:localhost:6443  -F vsshconfig k8s0`
3. Copy the k8s0:/root/.kube/config to ~/.kube/vagrant.config on your Mac and edit the vagrant.config file changing the line that has:
`server: https://10.0.100.11:6443` to `server: https://k8s0:16443` (this is necessary to match the cert used and forwarded port).  
4. Next, modify your local /etc/hosts adding k8s0 to the localhost line.  
5. Then run the following to load the config:
`export KUBECONFIG=~/.kube/vagrant.config`

You can now run native kubectl commands (assuming you have them installed - if not `brew install kubectl`) interacting with your multi-node cluster!
