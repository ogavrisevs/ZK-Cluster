
Introduction
-------------

This is proof of concept project. It demonstrates Zookeeper work in cluster (ensemble in zk terms). ZK is used as service discovery system. Nodes will be provisioned with Ansible and cluster durability will be tested. We will spin up multiple ZK nodes and one node for running Ansible in VirtualBox controlled by Vagrant.

Commands:
--------

Spinup ZK cluster, provision it and execute tests :

```
vagrant up
```

Destroy all VM's:

```
vagrant destroy
```

Run/ re-run provisioning on all nodes:

```
vagrant provision
```

You can also run comands on particular ( one ) node (see naming paragraph ):

```
vagrant provision ansible
```
 or
 ```
 vagrant destroy node1
 ```


Naming and URL's
---------


Host name     | IP        | Exhibitor                                                                            | Latency Test Results
------------- | ----------|--------------------------------------------------------------------------------------|--------------------------------------
node0  | 192.168.100.100  | [exhibitor/v1/ui/index.html](http://192.168.100.100:8080/exhibitor/v1/ui/index.html) |  [latencie_tests_results.log](http://192.168.100.100/latencie_tests_results.log)
node1  | 192.168.100.101  | [exhibitor/v1/ui/index.html](http://192.168.100.101:8080/exhibitor/v1/ui/index.html) |  [latencie_tests_results.log](http://192.168.100.101/latencie_tests_results.log)
node2  | 192.168.100.102  | [exhibitor/v1/ui/index.html](http://192.168.100.102:8080/exhibitor/v1/ui/index.html) |  [latencie_tests_results.log](http://192.168.100.102/latencie_tests_results.log)
. . .  | . . .            | . . .                                                                                | . . .
ansible| 192.168.100.200

Testing
--------
When ZK nodes are provisioned with Zookeeper and Exhibitor, provisioning will run smoke tests and latency tests using [zk-smoketest](https://github.com/phunt/zk-smoketest) suit. If tests will fail also  provisioning will fail. Smoke test results can be found in stdout logs. Latency test results will be published on Appache web server (see Naming paragraph for url ).

Zookeeper setup
----------------

We will setup cluster of 3-9 nodes, node count can be specified with `instance_count` variable in "Vagrantfile". Zookeeper documentation recommends to chose [2xF+1](https://zookeeper.apache.org/doc/trunk/zookeeperAdmin.html#sc_CrossMachineRequirements) count of nodes. Odd number of nodes will help cluster to be falt tolerant (for example in case with three ZK node cluster it can handle failure of one node). We will use [Netflix exhibitor](https://github.com/Netflix/exhibitor) to control ZK work on nodes it will also give Web UI. We will share configuration across nodes with Vagrant shared folders. In production it can be replaced with AWS.S3 bucket.

![FileStorage](https://github.com/Netflix/exhibitor/wiki/assets/file-config.png
)

Vagrant setup
--------------

Unfortunately there is no [officieal vagrant box](https://bugs.centos.org/view.php?id=6365#bugnotes
) from Centos so we will us one from [vagrantbox.es](http://www.vagrantbox.es) -> [CentOS 6.5 x86_64](https://github.com/2creatives/vagrant-centos/releases/tag/v6.5.3). Just 280 Mb and no extra packages preinstalled.


Netflix exhibitor
------------------

Exhibitor wiki page states:

>For the standalone version, copy exhibitor-xxx.jar to each server running ZooKeeper server. Use init.d or something similar to enable auto-run of Exhibitor.

but officially provided artifacts are missing main class and missing dependent libraries in jar. So we will build or own `true` fat-jar from [forked source code](https://github.com/ogavrisevs/exhibitor)
and store it in [Bintray repo ](https://bintray.com/ogavrisevs/maven/Exhibitor/1.5.1).


Ansible
-------

To better control and test Ansible provisioning we will not use Vagrant built-in ansible provisioning but instead spinup additional VM which will be used as Ansible host. We will reuse VM build in users and private/keys  :

* 'vagrant' user (with sudo right)
* vagrant [private key](https://github.com/mitchellh/vagrant/tree/master/keys
)


Also we will reuse [jdk install role](https://github.com/geerlingguy/ansible-role-java) from Ansible Galaxy. And will use ansible module [maven_artifact](http://docs.ansible.com/maven_artifact_module.html) which will be released only in version 2.0 (we will load it hacky way :D ).




