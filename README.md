# escacha - Split Brain Simulation Tool 
**escacha** is a tool designed for simulating connection splits between sites or nodes in a cluster environment. It's intended to be used in Grid Infrastructure environments thought it could be used with little adaptations with any other cluster stack. 
It begins with a variable definition section where we can specify:

* **MODOTEST**. [ 1 | 0 ] Determines if we want to perform a real split or just test our configuration and see what will be done
* **STORAGESPLIT**. [ 1 | 0 ] Include or exclude storage split test
* **NETWORKSPLIT**. [ 1 | 0 ] Include or exclude network split test
* **ONLINEDISKS**. [ 1 | 0 ] Once split test has finished we can execute an ASM online disks for all mounted DGs. Useful to leave the cluster in a healthy state and avoid disks being dropped
* **STRSCSI**. String used to identify disks in our configuration in /dev/disk/by-id. At the moment only tested on RHEL7 with SCSI disks with "scsi-" header. Useful for future different configurations
* **CABINASITE1** . String used to identify the fixed part of the WWIDs for LUNs in storage array in site1
* **CABINASITE2** . String used to identify the fixed part of the WWIDs for LUNs in storage array in site2
* **DELAY**. Seconds to wait before performing the split. This gives us time to check the basic information about cluster configuration and abort if we find any problem
* **CRSRESTART**. [ 1 | 0 ] We can perform cluster restart in the local node (crsctl stop crs / crsctl start crs). Useful when we expect issues in the split to restore healthy situation after restoring disk and network visibility.
* **ARUMON**. Path where arumon tool is deployed. arumon will be executed to get periodic cluster information during the split.
* **NODOSSITE1**. Space separated list of cluster nodes located in site1.
* **NODOSSITE2**. Space separated list of clueter nodes located in site2.
* **REDES**. Space separated list of interconnect network suffixes for building private hostnames for network split purposes
* **REBALANCEPOWER**. [ 0..1024 ] Power to use for ASM rebalance operations. Used in conjunction with ONLINEDISKS.

escacha is configured in this repo to be deployed using ansible. An example of how to deploy it to a configured host group is:

ansible-playbook -i hosts_oracle --limit environment-ora-pro playbooks/pb_environment_escacha.yml -v

It is recommended to use a playbook where arumon is also deployed. Currently arumon and escacha are separated products, but escacha is tighly copupled to arumon.

**Sample deployment ansible command:**

Exemplo para despregar a ferramenta no cluster de producion de tauli.

1. Configurar variables en **group_vars/grupo/grupo_escacha.yml**. Por exemplo, para tauli produción, group_vars/tauli-ora-pro/tauli-ora-pro-escacha.yml
1. Despregar con Ansible a aplicacion nos servidores do grupo correspondente. De novo para tauli produción:
ansible-playbook --limit tauli-ora-pro play_escacha.yml
1. Lanzar escacha para simular un split en modo real ou en modo test segundo a configuracion que teñamos aplicado.
ansible-playbook --limit tauli-ora-pro play_startescacha.yml
1. Parar escacha e, en funcion da configuracion, recuperar a normalidade no clúster.
ansible-playbook --limit tauli-ora-pro play_stopescacha.yml

O playbook **playescacha.yml** desprega tamen a ferramenta arumon. En caso de non querer despregarse (non saberia o motivo polo que facelo), debemos comentar a correspondente entrada no playbook ou crear un playbook novo.
