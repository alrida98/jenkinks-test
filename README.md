# Ansible-web-app

Configuration :

# 1. Master server and one or more worker node

On Master node :

Install Jenkins:

```
sudo apt update
sudo apt install openjdk-21-jdk -y
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc   https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]"   https://pkg.jenkins.io/debian-stable binary/ | sudo tee   /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
sudo systemctl status jenkins
```

Install Python & Ansible:

```
apt install software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install python3.11
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.11 1
sudo apt-add-repository ppa:ansible/ansible
sudo apt install ansible-core
sudo apt update
```

# 2. Confirm ansible is installed on the master server:

```
 ansible --version
```


# 3. SSH connectivity 

Make sure master node can SSH to the worker node 

``` ssh-keygen 

ssh-keygen
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/id_ed25519
Your public key has been saved in /home/ubuntu/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:cYspT584KW+VwDOZw4Fmsl1c6WdNEaMVIgcxU5uc/CY ubuntu@ip-172-31-39-36
The key's randomart image is:
+--[ED25519 256]--+
|       o .B=+ *+ |
|    . + + .B B.. |
|     * +.=. Bo   |
|    . . X=..o..  |
|      . S=.+E o  |
|       + +o. o   |
|      . =.o      |
|       o..       |
|       ..        |
+----[SHA256]-----+

```

Copy the key of the master and paste it into the worker nodes:

```
 cat id_ed25519.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMBJjOa4qPMV6Y7Pi+pnUkzZ3SuR3T1wWCbmnTUYIRR3 ubuntu@ip-172-31-39-36

```

Add this in authorized keys of the worker node  

```
$ cat authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMBJjOa4qPMV6Y7Pi+pnUkzZ3SuR3T1wWCbmnTUYIRR3 ubuntu@ip-172-31-39-36
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDhdywFJ+vqiuJGTCX8H8NMA8aI5sIKNJXSHfizJr42Xb5FQqnthq88GilbMeJXIRhOcY8PZv9WkvICa9/KzIIQYpphjK/bAIcqe5IiTGZjYXvZZ5Bukf1mqkdxRgyvgA1tdafV0Zvo52Plz/f/iMChFTfPNYAZiHg1qxF0m59RATzHRXYGdsI9kohAB0n834wE7wUjhbLJjdvKZx3olcTqgkaiACQd1WJHuicrtyeK425LeZQ7DyR8M4gNAHL1Ol+tHn9x4qdr/VVWtZVndIDZK3q0y+Vj5GtEN/p/ZPoZCYNMbXX2fhz/d7EAgDwPOvDExZmoUbrd1bY041AHcDcr test123

```

We should be able to SSH from Master to worker node : using ssh <Private_IP>


# 4. Access Jenkins on web UI , add Ansible Plugin and Tools as below 

Manage Jenkins --> Plugins --> Avaialable Plugins --> Ansible - Install the Plugin 

![image](https://github.com/user-attachments/assets/a767bddc-c58f-42c6-9b86-78b4230a8652)

In the Ansible tool : add the path of the Ansible installation from the  master server where we can execute the Ansible based commands , here it is /usr/bin

![image](https://github.com/user-attachments/assets/d2b13d23-a63e-4210-aa7c-16245d518c77)

# The project here has the folllowing files: Hosts file , Jenkinsfile , web-app.yml 

Hosts file :

* This file consists of the ansible worker node IP  and ansible_python_intepreter file 

```
[my-servers]
server1 ansible_host=172.31.34.105
#server2 ansible_host=172.31.83.39

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=ubuntu
```

* Here we use the private IP of the ansible_host 

Ansible playbook: 

* web-app.yml: The sample web application will be deployed on top of the Nginx application.

# Push the code on Github, Integrate Jenkins to create a Pipeline for CI/CD :

JenkinsFile

```
pipeline {
    agent any 
    stages {
      stage('SCM Checkout') {
        steps {
           checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/alrida98/jenkinks-test.git']]])  
        }
      }  
      stage('Execute Ansible') {
        steps {
           ansiblePlaybook credentialsId: 'ubuntu-slaves-key', disableHostKeyChecking: true, installation: 'ansible', inventory: 'hosts', playbook: 'web-app.yml'
        }  
      }
    }
}
```

* The first stage is SCM checkout : Connect to the Git repository which has the ansible playbook 
* Second step : Execute ansible-> deploy webapp , we can create the steps from UI as follows 
* Save the ubuntu-slaves-key details of the worker nodes , which is the pem key to authenticate with the Ubuntu worker node in this case 

![image](https://github.com/user-attachments/assets/f237f831-980f-4f0e-aeb1-083517297343)

 * Generate Pipeline for executing the ansible playbook: 

![image](https://github.com/user-attachments/assets/271d63f0-5032-4223-a086-0f86cddc1b81)

![image](https://github.com/user-attachments/assets/2460df59-0c00-45c0-90c7-b125a3053b9b)


* Save the JenkinsFile in the git repository and use the Pipeline script from SCM :

![image](https://github.com/user-attachments/assets/a00b7779-2ba2-4282-8bcb-96ef2b80aa21)


* The "Pipeline script from SCM" option in Jenkins refers to a way of defining and managing Jenkins pipeline scripts directly from a source code management (SCM) system, such as Git, instead of writing the pipeline script directly inside the Jenkins job configuration.

# 5. Save and Build the pipeline

* Atatched console output , shows us that we have installed the web app on nginx 

* Access the worker node public IP , you will see the web application setup 

![image](https://github.com/user-attachments/assets/8ed2140a-002b-41ef-aaeb-99845f400a6f)


