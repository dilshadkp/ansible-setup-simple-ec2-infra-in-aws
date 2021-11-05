# Ansible Playbook to setup a simple EC2 infra in AWS

## Description

This is a simple Ansible Playbook to launch an AWS EC2 instace along with a new key pair and a security group.

This Playbook will run in the Ansible Master machine itself(localhost) since this task do not involves any remove clients.

You need to put values for below variables in varialbes.vars file as per your requirement.

- **region: ""**
- **server_name: ""**
- **type: ""**
- **ami_id: ""**
- **key_name: ""**
- **sg_name: ""**


## Prerequisites

- Install Ansible in Ansible Master server. [Click here for Ansible Installation steps](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

 ## Ansible Modules used in this Playbook
- [ec2_key](https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_key_module.html)
- [copy](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html)
- [ec2_group](https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_group_module.html)
- [ec2](https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_module.html)
- [debug](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html)

 ## Tasks defined in the Playbook

 #### Task 1

>This task will create a new SSH Keypair in the given region with given name.
>Also, stores the output of this task to a variable ***key_info***

```python
    - name: "SSH keypair creation. Keyname is {{key_name}}"
      ec2_key:
        region: "{{region}}"
        name: "{{key_name}}"
        state: present
      register: "key_info"
```

 #### Task 2

>Copy the private key(pem) of the created keypair to the current path of Ansible Master machine as name {{key_name}}.pem.
>Then change the permissions of key file to *0400(r--------)*
>This task will run only if a new keypair is created.

```python
    - name: "copy private key to local server"
      when: key_info.changed == true
      copy:
        content: "{{key_info.key.private_key}}"
        dest: "./{{key_name}}.pem"
        mode: "0400"
```
 #### Task 3

>This task will create security group with inbound rules to open ports 80.443 and 22 to all IPv4 nad IPv6 IPs and outbound rules to open all ports to all IPv4 and IPv6 IPs.
>Also, stores the output of this task to a variable ***sg_info***
>If you want, you can modify or add ports to the security groups by modifying below task.

```python
    - name: "Create Security group"
      ec2_group:
        region: "{{region}}"
        name: "{{sg_name}}"
        description: "secuirty group for devops instance"
        tags:
          Name: "{{server_name}}-sg"          
        rules:
          - proto: tcp
            ports:
              - 80
              - 443
              - 22
            cidr_ip: 0.0.0.0/0
            cidr_ipv6: ::/0
            rule_desc: "allow all on ports 80, 443 and 22"
        rules_egress:
          - proto: -1
            ports: -1
            cidr_ip: 0.0.0.0/0
            cidr_ipv6: ::/0
      register: "sg_info"

```

 #### Task 4

>This task will create an EC2 instance in the given region with given AMI, instance type.
>Public key of above created SSH keypair will be added to this EC2 instance.
>Above created Security Group will be attached to this EC2 instance.
>Also, stores the output of this task to a variable ***ec2_info***

```python
    - name: "Create EC2"
      ec2:
        region: "{{region}}"
        key_name: "{{key_info.key.name}}"
        instance_type: "{{type}}"
        image: "{{ami_id}}"
        wait: yes
        group_id: "{{sg_info.group_id}}"
        instance_tags:
          Name: "{{server_name}}"
        count_tag:
          Name: "{{server_name}}"
        exact_count: 1
      register: ec2_info
```
 #### Task 5

>This task will print the Public IP along with the instance ID of the created EC2 instance.

```python
    - name: "Print Public IP of Instance"
      debug:
        msg: "PUBLIC IP OF {{ec2_info.tagged_instances[0].id}} : {{ec2_info.tagged_instances[0].public_ip}}"
```

 ## Execution
 - Put the Playbook in the Ansible Master server working directory.
 - Run a syntax check
 ```bash
ansible-playbook main.yml --syntax-check
```
 - Execute the Playbook
 ```bash
ansible-playbook main.yml
```
 ## Sample Output

![](https://i.ibb.co/QKLHrMZ/execution.png)

x------------------x---------------------x---------------------x-------------------x--------------------x---------------------x-------------------x

### ⚙️ Connect with Me 

<p align="center">
<a href="mailto:dilshad.lalu@gmail.com"><img src="https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white"/></a>
<a href="https://www.linkedin.com/in/dilshadkp/"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/></a> 
<a href="https://www.instagram.com/dilshad_a.k.a_lalu/"><img src="https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white"/></a>
<a href="https://wa.me/%2B919567344212?text=This%20message%20from%20GitHub."><img src="https://img.shields.io/badge/WhatsApp-25D366?style=for-the-badge&logo=whatsapp&logoColor=white"/></a><br />
