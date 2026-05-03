# Jenkins Installation

**Master Node :** 

sudo apt update

sudo apt install fontconfig openjdk-21-jre

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins

**Worker Node:** Only JDK will install

sudo apt update

sudo apt install fontconfig openjdk-21-jre

Now check Jenkins is install in Master :  systemctl status jenkins

sudo systemctl enable jenkins  ( On port 8080 jenkins running)

Enable 8080 Port in EC2

sudo cat /var/lib/jenkins/secrets/initialAdminPassword ( you will get password paste it in jenkin)

Login it 

Now how we connect Master and Agent Server. .?
Using SSH -> In master node  [cd ~/.ssh]
generate public and private key -> [ssh-keygen] using [cat pub] copy public ip.

Now go in agent server [cd ~/.ssh] [ls] and [vim authorized_keys] paste public ip in last.

**Make new agent**
Step 1. Create permanent agent.
Step 2. Remote root directory - > /home/ubuntu

Step 3. Labels -> archu (its means agent-any ke pace par agent {label archu} use karenge

Launch method -> Launch agent via SSH

Host - > agent ip address put it.

Credentials ? - click on [Add] slect [Kind] select ->  [ssh username with private key] , ID -> [any name like ubuntu-key], Username - [ubuntu], private key -> enterdirectly [ paste private key ]

Step 4. Host key verification strategy ?    -> Non verifying verification Strategy

Step 5. save

agent sucessfully connect and online.


**Run this agent in declarative pipeline**
```
pipeline {
   agent { label: "archu" }
   stages{
     stage('Hello'){
       steps{
              echo 'Hello Dosto'
       }
    }
}
   stages{
     stage('Create Folder'){
       steps{
         sh "mkdir -p devops"
       }
    }
}
   stages{
     stage('Bye'){
       steps{
      }
   }
}

}




