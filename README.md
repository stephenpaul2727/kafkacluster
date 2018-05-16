# kafkacluster-vbox
Ansible Scripts to automatically deploy a kafka Cluster and a zookeeper Quorum in your local machine VM's (Ubuntu flavored Linux) using VirtualBox


## Description
If you want to test a kafka cluster in development, play around with a kafkacluster and a zookeeper quorum, write a java application and integrate kafka streams etc. This Project would come in handy and would bootstrap a kafka cluster within minutes in your local development environment.

## Prerequisites
1. This Project assumes that you have Oracle VirtualBox installed in your local machine.
2. This Project assumes that you have installed Red Hat's Ansible for automation. Ansible is a python-based project and can easily be installed on your machine through pip/apt/yum etc.
3. The VMs you make will be using an Ubuntu Linux iso. please download the ISO file into your local machine.


## Configuring VM's
1. Start a VM in virtualbox with the ubuntu ISO downloaded from internet. Install ubuntu with username **ubuntu**.
2. Power off the VM and go to its settings. Add **Host Only Adapter** in addition to **NAT** in network section.
3. Power on the VM and configure a static ip by going to `/etc/network/interfaces`. Add a static ip by appending the following code to the end of the existing file:
```
# static ip
auto enp0s8
iface enp0s8 inet static
address 192.168.56.96
netmask 255.255.255.0
```
4. The address should picked from unused ip's in the enp0s8 interface range. I have 56.96 free. So picked it. you can pick anything else. Just remember to update the ip in the hosts file and group_vars/all of this project.
5. Add the VM's username (ubuntu) as a sudoer in /etc/sudoers list. just append the following line `ubuntu	ALL=NOPASSWD:ALL`. This is necessary for the user **ubuntu** to elevate his privileges during the automation process if necessary. 
6. Install openssh-server service using the command `sudo apt-get install openssh-server`. This is required as Ansible uses SSH protocol to automate tasks.
7. Reboot VM and ensure the static ip is active by checking enp0s8 interface inet in `ifconfig` result.
8. Power off VM
9. Create two clones of this VM you just made.
10. Boot up the new VMS and change their hostname in `/etc/hostname` and `/etc/hosts` to a unique name different from other VM's. 
11. Go to `/etc/network/interfaces` and choose an another unused IP in the enp0s8 interface ip range which is unique. Update it in the file and also remember to update hosts file and group_vars/all file in this project.
12. Reboot VM
13. After all the VM's are configured properly with SSH installed and unique static ip's. Now you are almost ready to run ansible scripts.
14. But before, please power off all VM's and start all of them in a `headless start` way. Now login into these VM's from your local machine using SSH.
15. Add your local machine public key to the remote machine's `~/.ssh/authorized_keys file. if it doesn't exist, create one.
16. Now exit the connection and login again using ssh. it should from now on let you authenticate into VM without password prompt. Also, please remove any password for your local machine public key. It would be a hassle to type password all the time. For instance, you can change it by typing the command `ssh-keygen -p` in mac OS.
17. Now power of all the VM's and go to settings. Add external SATA for your VM in `settings>storage`. Add disk size of 2GB. Ensure that this process is done for all the three VM's
18. This is necessary for kafka logs as they can get really large in a small amount of time.
19. Now you are all set to deploy kafka cluster into your three VM's.

## Deploying Kafka Cluster and Zookeeper Quorum
1. Download the project and go to its root directory.
2. Before you run any commands, please ensure that hosts file in root project directory is updated with the static ip's of the VM's.
3. Also ensure that the `group_vars/all` is also updated with the VM's ip's respectively.
4. That's it. Run this command `ansible-playbook -i hosts site.yml`
5. It will run all the roles starting with common configuration, then zookeeper quorum configuration, then kafka cluster configuration. 
6. Now you have a kafka cluster deployed into your VM's

> This project deploys a zookeeper quorum of 3 nodes and a kafka cluster into three VM's. This setup is okay for development purpose to save resources on your local machine. But for production use different servers for zookeeper and different servers for kafka. Also ensure that they are in different regions to ensure resiliency and reliability.

## Testing
1. Log into all the three VM's using three SSH windows from your local machine.
2. Open a fourth SSH window to any of the VM for observing logs. run `tail -100f /home/ubuntu//kafka/logs/server.log` to observe magic of kafka.
3. Run `sudo service kafka start` on all the VM's to start kafka cluster.
4. You can now create topics, partitions etc using console commands or API calls. Please refer to kafka docs.



