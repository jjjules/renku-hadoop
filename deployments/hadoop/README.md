# Hadoop deployment

We deploy Hadoop on multiple nodes using Ambari, a software from Cloudera that facilitates the installation. The nodes are separated between masters and slaves. There is no minimum number of nodes required, it is possible to deploy Hadoop on a single node containing all the master and slave processes but it should only be used for evaluation purposes. Typically the minimum number of nodes recommended for a minimum cluster is at . For more information on the number of nodes required for your use and their hardware recommendation, you can refer to ![this page](https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.1.4/cluster-planning/content/hardware-recommendations.html). ![As Ambari is no longer free](https://community.cloudera.com/t5/Support-Questions/How-do-I-download-the-latest-version-of-Ambari-and-HDP/m-p/299115/highlight/true#M219531), to avoid the subscription we use open source repositories. The ![Make Open Source Great Again (MOSGA) site](http://makeopensourcegreatagain.com/) has the repositories necessary for `CentOS 7` (we used `CentOS Linux release 7.9.2009 (Core)`). For the installation of Ambari you’ll have to choose one node as the Ambari server and the others will be Ambari agent. We describe how to set up the nodes so that the server install the modules automatically on all agents. Make sure to check ![the official instructions](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.4.0/bk_ambari-installation/content/ch_Getting_Ready.html) as the following instructions are not exhaustive but rather a user friendly application. For more information, you can also check the switch engine configuration we used to deploy under ![switchengines-configuration/roles](./switchengines-configuration/roles).

1. Install the package dependencies: `rpm`, `scp`, `curl`, `unzip`, `tar`, `wget`, `gcc`, OpenSSL (v1.01, build 16 or later), Python 2.7.12 (with python-devel)
1. Set the maximum number of open file descriptors to a sensible default (e.g. 10000). You can for example set a hard limit in the file `/etc/security/limits.conf`.
1. Set up password-less ssh connection between the Ambari server and the agents: generate a private and public SSH keys on the server in the root account. Copy the generated public key to each agent by appending the content of `/root/.ssh/id_rsa.pub` to the file `/root/.ssh/authorized_keys` on each host. Then manually connect to each agent from the host to ensure it is working and validate the authenticity of each host.
1. Enable NTP on all nodes: install `ntp` and enable it using `systemctl enable ntpd`
1. Disable ipv6 on all nodes. You can refer to the switch engine configuration to see how to disable it.
1. Set up the files `/etc/hostname` and `/etc/hosts` for local DNS resolution on all nodes. The first file should contain the Fully Qualified Domain Name (FQDN) and the second should have entries for the current node and then one line per node in the form <Local ip address> <domain name> FQDN.
1. Disable SELinux on all nodes with the command `setenforce 0`. You can also set `SELINUX=disabled` in `/etc/selinux/config` to ensure it will stay disabled after rebooting the machines.
1. On each nodes add the following MOSGA repositories to yum: http://makeopensourcegreatagain.com/repos/centos/7/hdp/ and http://makeopensourcegreatagain.com/repos/centos/7/ambari/2.7.5.0/.
1. Install `ambari-server` with yum on the Ambari server.
1. Setup the ambari server using `ambari-server setup` (if you encounter problems check the official documentation).
1. Start the Ambari server: `ambari-server start` and then you should be able to access the Ambari wizard by pasting the public ip address of your Ambari server in your browser. Again if you have trouble with this step don’t hesitate to check ![the official documentation](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.4.0/bk_ambari-installation/content/ch_Deploy_and_Configure_a_HDP_Cluster.html). You may have to add a certificate to the Ambari server to be able to access it from the browser. All remaining steps are well explained in the wizard: name your cluster, choose the services you want to install, assign the nodes, etc. It is important to use the root account for the wizard and when you are prompted for an ssh-key make sure to copy the private ssh key that has been generated on the Ambari server.