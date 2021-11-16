# Scripts
Project 1 scripts
Azure Elk Deployment

The files in this repository were used to configure the network depicted below.

Project 1: ReadTeam Diagram

 ![alt text](https://github.com/juliexo/Scripts/blob/main/diagrams/Red_Team_Diagram.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the YML and Config file may be used to install only certain pieces of it, such as Filebeat.

<p>
  <details>
    <summary>Ansible Elk Installation Playbook</summary>
<pre><code>---
- name: Configure ELK
  hosts: elk
  remote_user: ELKAdmin
  become: True
  tasks:

  - name: use more memory
    sysctl:
      name: vm.max_map_count
      value: "262144"
      state: present
      reload: yes

  - name: docker.io
    apt:
      update_cache: yes
      name: docker.io
      state: present

  - name: Install pip3
    apt:
      force_apt_get: yes
      name: python3-pip
      state: present

  - name: install python module
    pip:
      name: docker
      state: present

  - name: elk container
    docker_container:
      name: elk
      image: sebp/elk:761
      state: started
      restart_policy: always
      published_ports:
                - 5601:5601
                - 9200:9200
                - 5044:5044

  - name: Enable service docker on boot
    systemd:
      name: docker
      enabled: yes
</code></pre>


  </details>
  </p>
  

<p>
 <details>
  <summary>DVWA Install File</summary>
  <pre><code>---
- name: Config Web VM with Docker
  hosts: webservers
  become: true
  tasks:
  - name: docker.io
    apt:
      force_apt_get: yes
      update_cache: yes
      name: docker.io
      state: present

  - name: Install pip3
    apt:
      force_apt_get: yes
      name: python3-pip
      state: present

  - name: Install Docker python module
    pip:
      name: docker
      state: present
  - name: download and launch a docker web container
    docker_container:
      name: dvwa
      image: cyberxsecurity/dvwa
      state: started
      restart_policy: always
      published_ports: 80:80

  - name: Enable docker service
    systemd:
      name: docker
      enabled: yes
      </code></pre>
 </details>
 </p>
 
 
<p>
 <details>
  <summary>Filebeat Playbook</summary>
<pre><code>---
- name: Installing and Launch Filebeat
  hosts: webservers
  become: yes
  tasks:

  - name: Download filebeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

  - name: Install filebeat .deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

  - name: Drop in filebeat.yml
    copy:
      src: /etc/ansible/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml


  - name: Enable and Configure System Module
    command: filebeat modules enable system

  - name: Setup filebeat
    command: filebeat setup

  - name: Start filebeat service
    command: service filebeat start

  - name: Enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes
      </code></pre>
 </details>
 </p>
 
 <p>
 <details>
  <summary>Metricbeat Playbook</summary>
<pre><code>---
- name: Install metric beat
  hosts: webservers
  become: true
  tasks:

  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

    # Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.4.0-amd64.deb

    # Use copy module
  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

    # Use command module
  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker

    # Use command module
  - name: setup metric beat
    command: metricbeat setup

    # Use command module
  - name: start metric beat
    command: service metricbeat start

    # Use systemd module
  - name: Enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes
      </code></pre>
 </details>
 </p>
 
This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting traffic to the network.

What aspect of security do load balancers protect? What is the advantage of a jump box?

Load balancers protect the system from DDoS attacks by shifting attack traffic.  A Jump box gives a user access from a single node that can be secured and monitored.


Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the data and system logs.

What does Filebeat watch for?

The filebeat watches for any information in the file system that has been changed and when the change occurred.

What does Metricbeat record?

Metricbeat takes metrics and statistics collected from the server and sends them to the output selected by the user.

The configuration details of each machine may be found below.
| Name Of VM | Function | IP:Private | IP:Public |	OS |
| --- | :---: | :---: | :---: | :---: |
| RedTeamVM1 |	Gateway |	10.2.0.4 |	52.249.217.162 | Linux |
| Web 1 | Web Server	| 10.2.0.5	| Static	| Linux |
| Web 2 | Web Server	| 10.2.0.6	| Static	| Linux |
| RedTeamLB | Load Balancer | Static | Static	| Linux |
| Elk-VM |	Elk Server |	10.3.0.5 |	40.83.172.96 |	Linux |
| Workstation	| Access | Control	| External	| External	| Windows 10 |





Access Policies 

The machines on the internal network are not exposed to the public Internet. 

Only the ELK Virtual machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
Whitelisted IP addresses:

-	Any Public IP through TCP 5601

Machines within the network can only be accessed by Workstation and Jump-box-provisioner.

Which machine did you allow to access your ELK VM? What was its IP address?_

-	RedTeamVM1: 10.2.0.4 via SHH port 22
-	Workstation Public IP via TCP 5601

A summary of the access policies in place can be found in the table below.

| Name |	Publicly Accessible |	Allowed IP Address |
| --- | :---: | ---: |
| RedTeamVM1	| No	| Workstation Public IP on SSH 22 |
| Web 1 |	No	| 10.2.0.5 via SSH 22 |
| Web 2 |	No |	10.2.0.6 via SSH 22 |
| ELK-VM/ELK Server	| No	| Workstation Public IP on TCP 5601 |
| Load Balancer	| No |	Workstation Public IP on HTTP 80 |

Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because Ansible allows you to quickly and easily deploy applications. 

What is the main advantage of automating configuration with Ansible?

You do not need to write custom scripts to automate your system or deployments. Ansible requires you to cerate explained tasks via playbook.yml files and then deploys them to all of your machines that are designated within the host file. Ansible also figures out how to get each system to your desired state through the playbook instructions.



The playbook implements the following tasks:

-	Machine groups and Remote User Specifications 

-  name: Config ELK VM with Docker
  	  hosts: elk
  	  remote_user: ELKAdmin
  	  become: true
      tasks:

-	Increase System Memory

-  name: Use more memory
    sysctl:
    name: vm.max_map_count
    value: '262144'
    state: present
    reload: yes

-	Install the following services:
o	Docker.io
o	Python3-pip
o	Docker elk container
-	Launching and exposing the container with these published ports:
o	5601:5601
o	9200:9200
o	5044:5044

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

Elk Server
![alt text](https://github.com/juliexo/Scripts/blob/main/images/ELKServerScreenshot.png)  

Web-1
![alt text](https://github.com/juliexo/Scripts/blob/main/images/Web1Screenshot.png)
 

Web-2
![alt text](https://github.com/juliexo/Scripts/blob/main/images/Web2Screenshot.png)




Target Machines & Beats

This ELK server is configured to monitor the following machines:

List the IP addresses of the machines you are monitoring:

-	Web-1: 10.2.0.5
-	Web-2: 10.2.0.6

We have installed the following Beats on these machines:

-	Elk Server
-	Web-1
-	Web-2

Specify which Beats you successfully installed:

-	Filebeat
-	Metricbeat

These Beats allow us to collect the following information from each machine:

Filebeat allows us to collect log events with files generated by Apache or by using MS Azure Tools.

Metricbeat allows us to collect metrics and statistics through numerical outputs like CPU usage.

Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:

-	Copy the Elk Server Config file to /etc/ansible
![alt text](https://github.com/juliexo/Scripts/blob/main/images/ELKList.png)


-	Update the hosts file to include address of your webservers and ELK Private IP address:
![alt text](https://github.com/juliexo/Scripts/blob/main/images/Hostfile.png)

-	Update the filebeat-config.yml and the metricbeat-config.yml to designate the address of the ELK server:
o	output.elasticsearch:
o	output.kibana:

-	Filebeat-config 
![alt text](https://github.com/juliexo/Scripts/blob/main/images/filebeatconfig.png)
![alt text](https://github.com/juliexo/Scripts/blob/main/images/filebeatconfig2.png)




-	Metricbeat-config
![alt text](https://github.com/juliexo/Scripts/blob/main/images/metricbeatconfig.png)
![alt text](https://github.com/juliexo/Scripts/blob/main/images/metricbeatconfig2.png) 

-	Run the filebeat playbook, and navigate to kibana, under “add data logs/System_logs” to check that the installation worked as expected.  
![alt text](https://github.com/juliexo/Scripts/blob/main/images/modulestatus.png)
-	Run the metricbeat playbook, and navigate to Kibana under “add metric data/Docker” to check that the installation worked as expected.  
![alt text](https://github.com/juliexo/Scripts/blob/main/images/modulestatus2.png)
Which file is in the playbook?
-	filebeat-playbook.yml
-	metricbeat-playbook.yml
Where do you copy it?
-	/etc/ansible
Which file do you update to make Ansible run the playbook on a specific machine?
-	/etc/ansible/hosts and add the private IP addresses under the [WEBSERVERS] section or [ELK] section of the host file.
How do I specify which machine to install the ELK server on versus which to install filebeat on?
-	By specifying groups in the /etc/ansible/hosts file, labeled [WEBSERVERS] for filebeat, and [ELK] for ELK installation.
Which URL do you navigate to in order to check that the ELK server is running?
-	http:[Elk.VM.IP]:5601/app/kibana where the [Elk.VM.IP] is the Public IP address of the ELK server.

Terminal Commands Needed:
| Command |	Purpose |
| --- | :---: |
| SSH -I [name of keygen file] (user@ipaddress) |	Remote into your JumpBox Desktop
| SSH-Keygen	| Generate Public and Private keys |
| sudo apt install docker.io |	Install Docker |
| sudo service docker start |	Start the Docker service |
| sudo docker container list -a	| Show all containers services |
| sudo docker start <container_name> |	Start the container specified |
| sudo docker attach <container_name>	| Remote into the specified container |
| ansible -m ping all	| Check the connection of ansible containers |
| ansible-playbook <playbook.yml_file>	| Run a playbook.yml file |
| nano <name_of_playbook.yml>	| Create an Ansible playbook |
| sudo docker pull cyberxsecurity/ansible bash	| Run and create a docker image |
| sudo docker ps -a | List all active/inactive containers |


