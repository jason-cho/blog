

# PaaS + kube & Docker(2020.08.19~21)

## minishift installation

### Specification recommand

* openshift | minishift
* CPU AMD 8c => vCPU 16 | vCPU 8 
* RAM 32G | 16G
* SSD 500G | 256G

* 1st error 
 
	KEA@DESKTOP-1U7J9KN D:\minishift
	# minishift.exe start --cpus 4 --memory 8GB --disk-size 80GB --vm-driver virtualbox
	-- Starting profile 'minishift'
	-- Check if deprecated options are used ... OK
	-- Checking if https://github.com is reachable ... OK
	-- Checking if requested OpenShift version 'v3.11.0' is valid ... OK
	-- Checking if requested OpenShift version 'v3.11.0' is supported ... OK
	-- Checking if requested hypervisor 'virtualbox' is supported on this platform ... OK
	-- Checking if VirtualBox is installed ... OK
	-- Checking the ISO URL ... OK
	-- Downloading OpenShift binary 'oc' version 'v3.11.0'
	 52.98 MiB / 53.59 MiB [===================================================================================================================================>-]  98.88% 0s- 53.59 MiB / 53.59 MiB [=====================================================================================================================================] 100.00% 0sOK
	-- Checking if provided oc flags are supported ... OK
	-- Starting the OpenShift cluster using 'virtualbox' hypervisor ...
	-- Minishift VM will be configured with ...
	   Memory:    8 GB
	   vCPUs :    4
	   Disk size: 80 GB

	   Downloading ISO 'https://github.com/minishift/minishift-centos-iso/releases/download/v1.16.0/minishift-centos7.iso'
	 45.99 MiB / 370.00 MiB [=========================================>-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 370.00 MiB / 370.00 MiB [==================================================================================================================================================================================================================] 100.00% 0s -- Starting Minishift VM ..... FAIL E0818 11:49:45.397090    8756 start.go:494] Error starting the VM: Error creating the VM. Error creating machine: Error in driver during machine creation: open /Users/KEA/.minishift/cache/iso/centos/v1.16.0/minishift-centos7.iso: The system cannot find the path specified.. Retrying.
	Error starting the VM: Error creating the VM. Error creating machine: Error in driver during machine creation: open /Users/KEA/.minishift/cache/iso/centos/v1.16.0/minishift-centos7.iso: The system cannot find the path specified.

* 2nd error

	# minishift.exe start --cpus 4 --memory 8GB --disk-size 80GB --vm-driver virtualbox
	-- Starting profile 'minishift'
	-- Check if deprecated options are used ... OK
	-- Checking if https://github.com is reachable ... OK
	-- Checking if requested OpenShift version 'v3.11.0' is valid ... OK
	-- Checking if requested OpenShift version 'v3.11.0' is supported ... OK
	-- Checking if requested hypervisor 'virtualbox' is supported on this platform ... OK
	-- Checking if VirtualBox is installed ... OK
	-- Checking the ISO URL ... OK
	-- Checking if provided oc flags are supported ... OK
	-- Starting the OpenShift cluster using 'virtualbox' hypervisor ...
	-- Starting Minishift VM .... FAIL E0818 11:53:26.790880    1964 start.go:494] Error starting the VM: Error getting the state for host: machine does not exist. Retrying.
	Error starting the VM: Error getting the state for host: machine does not exist

* 3rd error

	KEA@DESKTOP-1U7J9KN D:\minishift
	# minishift.exe start --cpus 4 --memory 8GB --disk-size 80GB --vm-driver virtualbox
	-- Starting profile 'minishift'
	-- Check if deprecated options are used ... OK
	-- Checking if https://github.com is reachable ... OK
	-- Checking if requested OpenShift version 'v3.11.0' is valid ...
	   Hit github rate limit: GET https://api.github.com/repos/openshift/origin/releases/tags/v3.11.0: 403 API rate limit exceeded for 123.214.186.194. (But here's the good news: Authenticated requests get a higher rate limit. Check out the documentation for more details.) [rate reset in 45m06s]
	FAIL

## SOLUTONS

* minishift delete
* minishift.exe start --cpus 4 --memory 8GB --disk-size 80GB --vm-driver virtualbox



## VM installation

* Ansible_master1
 - master1 : 192.168.235.128
 - worker1 : 192.168.235.129
 - worker2 : 192.168.235.130
 

## add to /etc/hosts 

	192.168.235.128 master
	192.168.235.129 worker1
	192.168.235.130 worker2

	hostnamectl set-hostname master1
	hostnamectl set-hostname worker1
	hostnamectl set-hostname worker2


 


------------

# MISC

* (berryz webshare)[http://192.168.0.119]



# ETC - minishift


## inventory

	[myself]
	localhost

	[webserver]
	worker1
	worker2

## cat ansible.cfg
	[defaults]
	inventory = ./inventory


## nfs.yml


	[root@master ansible]# cat nfs.yml
	---
	- name: Setup for nfs server
	  hosts: localhost
	  gather_facts: no

	  tasks:
		- name: make nfs_shared directory
		  file:
			path: /home/devops/nfs_shared
			state: directory
			mode: 0777

		- name: configure /etc/exports
		  become: yes
		  lineinfile:
			path: /etc/exports
			line: /home/devops/nfs_shared 192.168.235.0/24(rw,sync)

		- name: Install NFS
		  become: yes
		  yum:
			name: nfs-utils
			state: present

		- name: nfs service start
		  become: yes
		  service:
			name: nfs-server
			state: restarted
			enabled: yes

		- name: nfs service open
		  become: yes
		  firewalld:
			service: nfs
			zone: public
			permanent: yes
			immediate: yes
			state: enabled

		- name: nfs service open 1
		  become: yes
		  firewalld:
			service: rpc-bind
			zone: public
			permanent: yes
			immediate: yes
			state: enabled

		- name: nfs service open 3
		  become: yes
		  firewalld:
			service: mountd
			zone: public
			permanent: yes
			immediate: yes
			state: enabled

		- name: nfs service open 2
		  become: yes
		  firewalld:
			port: 111/tcp
			zone: public
			permanent: yes
			immediate: yes
			state: enabled

	- name: Setup for nfs clients
	  hosts: webserver
	  gather_facts: no

	  tasks:
		- name: make nfs_client directory
		  file:
			path: /home/devops/nfs
			state: directory

		- name: Install NFS
		  become: yes
		  yum:
			name: nfs-utils
			state: present

		- name: mount point directory as client
		  become: yes
		  mount:
			path: /home/devops/nfs
			src: master:/home/devops/nfs_shared
			fstype: nfs
	#        opts: nfsvers=4
			state: mounted

