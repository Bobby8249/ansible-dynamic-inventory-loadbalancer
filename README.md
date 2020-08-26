# Ansible-task2
In this article, we configure the apache webserver httpd on AWS EC2 instance with ansible.


Task Description -
Statement: Deploy Web Server on AWS through ANSIBLE!

🔹 Launch an AWS instance with the help of ansible. 

🔹 Retrieve the public IP which is allocated to the launched instance. 

🔹 With the help of the retrieved Public IP configure the web server in the launched instance.

Before starting task Lets something know about Ansible & AWS -

Introduction Of ANSIBLE -

*Ansible is a tool that is used for Configuration Management (CM). In Ansible we only tell to "what to do". We don't need to tell "How to do" because Ansible already knows that "How to do this task" on each type of Operating System.

*This intelligence of Ansible comes from "modules". Modules know "How to do this operation on each OS". We don't require to tell particular OS command. Ansible doesn't perform any operation on the OS. It is done by respective OS commands. Ansible module knows which command is required to run on OS to do this.

Introduction Of AWS -

****AWS is a cloud that is provided by Amazon. Amazon Elastic Compute Cloud (Amazon EC2 ) is a part of Amazon.com's cloud-computing platform, Amazon Web Services, that allows users to rent virtual computers on which to run their own computer applications. AWS EC2 service provisions resources like RAM, HardDisk. CPU etc.

Prerequisite:-
Have an AWS account.
Created the IAM role.
Ansible installed.
To communicate with AWS we need -

> "boto" API

> Access key and Private key (To login in AWS)

Step-1)Install "boto" & "boto3" libraries -
This command installs boto3 in the Controller node.

pip3 install boto3
                                                                  AND

pip3 install boto







Add alt text
No alt text provided for this image
I already have an access key and private key to log in to AWS. if you don't have any key then create it through the AWS account.

Step - 2 Create Ansible Configuration File -







Add alt text
No alt text provided for this image
To Create an Ansible configuration file:

vim /etc/ansible/ansible.cfg

To See data of configuration file:

cat /etc/ansible/ansible.cfg

Data inside the ansible configuration file:

[defaults]
inventory = /myec2
host_key_checking = False
roles_path = /root/ansible_roles
private_key_file = /root/bobby_demo.pem
remote_user = ec2-user
ask_pass = false


[privilege_escalation]
become = true
become_user = root
become_method = sudo
become_ask_pass = false


I have already "bobby_demo.pem" key for ssh login in my system. If you don't have then put here.

Step-2)configure ansible.cfg for roles,dynamic inventory.
goto your os.
Configure Roles
make one workspace for roles.
mkdir /root/ansible_roles
Step-3)Configure Dynamic Inventory
Now create one more directory for dynamic inventory.
mkdir /myec2
Dynamic Inventory Directory -(In my case "/myec2")
For the dynamic inventory, download ec2.py and ec2.ini from this given URL, and paste in /myec2 folder:

To Download ec2.py by command line:
wget https://github.com/ansible/ansible/tree/stable-2.9/contrib/inventory/ec2.py
To Download ec2.ini by command line:
wget https://github.com/ansible/ansible/tree/stable-2.9/contrib/inventory/ec2.ini
Here you also need privilege escalation because in aws we have to configure all the configuration done by the user root only.
After this edit your ec2.py







Add alt text
No alt text provided for this image
change your python path in my case it is /usr/bin/python3.
Save it.
After this make this file executable by this following command:
chmod +x ec2.py
Step-3)Configure key for AWS ec2-instance
After that, you also need to copy the key.pem for the ec2 instance launch.
After copying your key, make it executable by this following command:
chmod 600 Key_Name.pem

Step-4) Now set these environment variables - 
"AWS_REGION " , "AWS_ACCESS_KEY_ID" , "AWS_SECRET_ACCESS_KEY"

you need to configure by your IAM user credentials so that we can commute the AWS cloud and launch ec2 instance.
Here we need to type some command to configure IAM user
export AWS_REGION='ap-south-1'
export AWS_ACCESS_KEY_ID='ACCESS_KEY'
export AWS_SECRET_ACCESS_KEY='SECRET_KEY'

Here u need to type region, AWS access key, AWS secret key provided by the IAM user.
To check your inventory is working fine then run this command "ansible all --list-hosts" - (I don't have any instance at this time)







Add alt text
No alt text provided for this image
Now It's working fine.

Step-5)Setup Ansible Role For Launch AWS EC2 Instance
the command for creating role:
ansible-galaxy init ec2
In my case, my role name is "ec2".
After go to this ec2 directory, you will find some files and directory.







Add alt text
No alt text provided for this image
Now we need to write a playbook for launching the ec2 instance.
Now Here we need to use the vault concept for providing AWS IAM credentials.
To know more about ansible vault visit this article -

Now got to vars folders and type:
ansible-vault create secure.yml








Add alt text
No alt text provided for this image
Type the password and confirm it.
Here u need to make variables for the access key and secret key so that we can use these variables in the task/main.yml file
Now open tasks/main.yml with a text editor.
In Ansible we can use ec2 module to launch aws ec2 instance, to know more about ec2 module visit this link -










Add alt text
No alt text provided for this image
Write variable for this task in "vars" directory (file "main.yml") 
Make sure your security group allows ssh (22) and HTTP(80), https(443) ports.
bobby_demo.pem is key-pair for aws ec2 instance.
In this playbook, I am retrieving the IP address of AWS ec2 instance and copying in inventory  "/myec2".
Code:
---
# tasks file for ec2
- name: launching AWS instance using Ansible
  ec2:
    key_name: "bobby_demo"
    instance_type: "{{ aws_instance_type }}"
    image: "{{ ami_id }}"
    wait: yes
    count: 1
    instance_tags:
      Name: "{{ aws_instance_tags }}"
    vpc_subnet_id: "{{ subnet_id }}"
    assign_public_ip: yes
    region: "{{ aws_region }}"
    state: present
    group_id: "{{  security_group_id  }}"
    aws_access_key: "{{ access_key }}"
    aws_secret_key: "{{ secret_key }}"


Here you can see I am using the ec2 module for launching ec2 instance, and I am using variables for different keywords, at last, I provided AWS access and secret key for authentication to my AWS account.
Now open vars/main.yml with a text editor, and type.







Add alt text
No alt text provided for this image
Code:
---
# vars file for ec2
aws_instance_type: "t2.micro"
ami_id: "ami-0ebc1ac48dfd14136"
aws_instance_tags: "ansbiletask2"
subnet_id: "subnet-4b493b07"
aws_region: "ap-south-1"
security_group_id: "sg-0a87cd4fc86b84bec"


Here you can see I using different variables for launching ec2 instance.
After this setup now makes one basic playbook file to run our ec2 roles.
File=> task2.yml
Code:
- hosts: localhost
  vars_files:
    - /root/ansible_roles/ec2/vars/secure.yml
  roles:
     
    - role: ec2

Here we need to run this file localhost and gave the path of vault file I,e aws.yml, and use role aws.
Now run this file, type.
ansible-playbook --ask-vault-pass task2.yml
It asks for a vault password Type.
Output:







Add alt text
No alt text provided for this image







Add alt text
No alt text provided for this image
Step-6) Setup Ansible Role for Configure webserver on EC2 instance.
Here we need to make a role for a webserver.
to make a role, go to your Role path, and type:
ansible-galaxy init webserver








Add alt text
No alt text provided for this image
Now open tasks/main.yml with a text editor and type Task for install Apache webserver







Add alt text
No alt text provided for this image
Code:
---
# tasks file for webserver


- name: httpdconfigration
  package:
       name: "{{ package_name }}"
       state: present
- name: install git
  package:
          name: "git"
          state: present
- name: start httpd
  service:
          name: "{{ package_name }}"
          state: started
          enabled: yes
- name: copy github code
  git:
          repo: "{{ git_repo }}"
          clone: yes
          dest: "{{ website }}"


Here I am again using multiple variables.
Here I am installing httpd by using package module.
After that starting and enabling services of httpd by using module => service.
For our website I use git, first, I install it and use the git module to copy our code.
Now after this open file vars/main.yml with text and type:







Add alt text
No alt text provided for this image
Code:
---
# vars file for webserver
package_name: "httpd"
git_repo: "https://github.com/Bobby8249/demo-webpage.git"
website: "/var/www/html"


Now here we need to type GitHub repo and use this variable in our tasks/main.yml file
After that make one small ansible-playbook to run our web server role.
Make on file, Name=> web.yml
- hosts: all
  roles:
          - role: webserver

Save it and run it, type.
ansbile-playbook web.yml
OUTPUT:
Copy the Public_ip of ec2 instance on the browser.







Add alt text
No alt text provided for this image







Add alt text
No alt text provided for this image







Add alt text
No alt text provided for this image
Our task is successfully completed.

Github link of  Website Code:


Github link For Ansible Code:


