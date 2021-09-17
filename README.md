# Ansible-Cloud-and-ELK
Cloud Security ELK and DVWA set up
## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![image](https://user-images.githubusercontent.com/85475926/133862263-60927d9e-679d-4add-9d8c-ee89f3e44674.png)




These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the playbook file may be used to install only certain pieces of it, such as Filebeat.

  Playbook 1: Pentest.yml
  ```---
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
      published_ports: 80:80

  - name: Enable docker service
    systemd:
      name: docker
      enabled: yes
```

  Playbook 2: elk.yml
  ```---
- name: Configure Elk VM with Docker
  hosts: elk
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present

    # Use apt module
    - name: Install python3-pip
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

    # Use pip module (It will default to pip3)
    - name: Install Docker module
      pip:
        name: docker
        state: present

    # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144

    # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: 262144
        state: present
        reload: yes
```
        
  Playbook 3: filebeat.yml
   ``` ---
- name: Installing and Launch Filebeat
  hosts: webservers
  become: yes
  tasks:
    # Use command module
  - name: Download filebeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb
    # Use command module
  - name: Install filebeat .deb
    command: dpkg -i filebeat-7.4.0-amd64.deb
    # Use copy module
  - name: Drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml
    # Use command module
  - name: Enable and Configure System Module
    command: filebeat modules enable system
    # Use command module
  - name: Setup filebeat
    command: filebeat setup
    # Use command module
  - name: Start filebeat service
    command: sudo service filebeat start
    # Use systemd module
  - name: Enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes
```
### This document contains the following details
 - Description of the Topology
 - Access Policies
 - Elk Configuration
 - Beats in Use
 - Machines Being Monitored
 - How to Use the Ansible Build
   

### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network.
-The advantage of a jump box is that it restricts public access, minimizing the possibility for attacks through configuration of the Network Security Group. Also, SSH connections to the jump box can be monitored and can easily identify unauthorized connections. 

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the configuration and system files.
-Filebeat monitors log files.
-Metricbeat collects operating system and service statistics from VMs.

The configuration details of each machine may be found below.

| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump Box | Gateway  | 10.1.0.4   | Linux            |
| Web1     | DVWA     | 10.1.0.5   | Linux            |
| Web2     | DVWA     | 10.1.0.6   | Linux            |
| ElkVM    | Elk      | 10.3.0.6   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box machine can accept SSH connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- 68.7.42.218

Machines within the network can only be accessed by the Jump Box.
- Jump Box can SSH into the ELK VM. Jump Box IP is 10.1.0.4

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Address | Allowed Ports |
|----------|---------------------|--------------------|---------------|
| Jump Box | Yes (SSH)           | 68.7.42.218        | 22            |
| Web 1    | Yes (HTTP)          | 68.7.42.218        | 80            |
| Web 2    | Yes (HTTP)          | 68.7.42.218        | 80            |
| ELK      | Yes (HTTP)          | 68.7.42.218        | 5601          |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- Building, configuring and deploying is quicker through ansible rather than doing so on a machine one by one.
- Controls Operating system, and software updates. 

The playbook implements the following tasks:
- The Pentest playbook sets up DVWA servers installing the following tasks: Docker, Python, Docker's Python Module, DVWA Docker container, and Enables Docker Service.
-  The Elk playbook was used to set up and launch the ELK repository server in a Docker Container, installing the following tasks: Docker, Python, Docker's Python Module, Increase Virtual memory to support ELK stack, Docker ELK container.
- Filebeat Playbook was used to deploy filebeat service on the web servers, allowing us to monitor them through our Elk VM. 

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![image](https://user-images.githubusercontent.com/85475926/133860796-7fface25-9d41-4ffb-80f3-ae17fc491858.png)


### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- Web 1: 10.1.0.5
- Web 2: 10.1.0.6

We have installed the following Beats on these machines:
- Filebeat
- Metricbeat

These Beats allow us to collect the following information from each machine:
- Filebeat collects and ships logs from Web 1 and Web 2 servers
- Metricbeat collects and ships our system metrics from the OS.

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the playbook files to an ansible container.
- Update the Ansible hosts file in ```/etc/ansible/hosts``` to include the following
```
[webservers]
#alpha.example.org
#beta.example.org
#192.168.1.100
#192.168.1.110
10.1.0.5 ansible_python_interpreter=/usr/bin/python3
10.1.0.6 ansible_python_interpreter=/usr/bin/python3

[elk]
10.3.0.6 ansible_python_interpreter=/usr/bin/python3
```
- Run the playbook, and navigate to your server's public IP address in a browser to check that the installation worked as expected.

- All files within my Ansible folder are playbooks.
- Update your Ansible hosts file to desired IP addresses. The playbook itself specifies which server filbeat and ELK will be deployed to.
- You can check to see if your Elk server works through http://"elk-server-ip":5601/app/kibana


