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
```

# Lets make DjangoCICD pipeline - declarative pipeline
Project - I want developer commit the code on GitHub webhook automatically trigger and git clone on jenkins 

Step 1. take any name of pipeline, GitHub project - paste URL, Build trigger -> Github hook trigger for GITscm polling

Step 2. script

```
pipeline{
   agent { label: "archu" }

   stages {
    stage("code"){
      steps{
        echo "This is cloning the code"
        git url: "URL" , branch: main
        echo "code clonnig sucessfull"
      }

    }
    stage(Build){
        steps{
          echo "This is cloning the Build"
          sh "whoami"
          sh "docker build -t notes-app:latest ."
        }

    }
    stage("Push to DockerHub"){
        steps{
          echo "This is Pushing image to docker hub"
          sh "Docker Login"
          sh "docker image tag notes-app:latest archanakidocker/notes-app:latest"
          sh "docker archanakidocker/push notes-app:latest
        }

    }
    stage("Deploy"){
        steps{
          echo "This is cloning the Deploy"
          sh "docker run -d -p 8000:8000 notes-app:latest"
        }

    }



   }

}



```

**Now I want to add pipeline use plugins -> add [pipeline stage view] plugins Install.**
**Docker Install on Agent terminal -> sudo apt-get install docker.io** docker ps will give you permission denied error.
**so, for that use command -> sudo usermod -aG docker $USER && newgrp docker (refresh my docker group)**
**8000 port add in EC2**


```
FROM python:3.9
WORKDIR /app/backend
COPY: requirement.txt /app/backend
RUN: pip install -r requirement.txt
COPY . /app/backend
EXPOSE 8080
CMD python /app/backend/manage.py runserver 0.0.0.0:8000

```

**You can also deploy your application through docker compose. why it need to deploy app on deocker compose?
because if u deploy ur app on 8000 and again you build your application then deployment will be failed.
allready running app on 8000.
**So avoid using docker run command**

```
stage("Deploy"){
        steps{
          echo "This is cloning the Deploy"
          sh "docker run -d -p 8000:8000 notes-app:latest"
        }

    }
**Replace to this**
    stage("Deploy"){
        steps{
          echo "This is cloning the Deploy"
          sh "docker compose up -d"
        }

    }

```


**Docker Compose**
```
version : "3.3"
services :
 web :
  build:
   context:
   ports :
     - "8000:8000"

```
**Docker compose must be install in agent -> sudo apt-get install docker-compose-v2**
**docker stop containerID && docker rm containerID**


# Credentials Binding : To hide my Credentials in environment variables
Go To Dashboard -> Manage Jenkins -> Credentials -> Stores Scoped to Jenkins (global) click -> add global credentials

Username - Archanakidocker
passsword - for password go to docker hub creating personal access tokens and paste token in password
ID- dockerHubCred
click on create
its means dockerHubCred name ki id hai and password hai

```
  stage("Push to DockerHub"){
        steps{
          echo "This is Pushing image to docker hub"
          sh "Docker Login"
          sh "docker image tag notes-app:latest archanakidocker/notes-app:latest"
          sh "docker archanakidocker/push notes-app:latest
        }

    }

Pushed to dockerHub

  stage("Push to DockerHub"){
        steps{
          echo "This is Pushing image to docker hub"
          withCredentials([usernamePassword(
                    credentialsId:"dockerHubCreds",
                    usernameVariable:"dockerHubUser", 
                    passwordVariable:"dockerHubPass")])
          sh "docker login -u ${env.dockerHubUser} p ${env.dockerHubPass}"          
          sh "docker image tag notes-app:latest ${env.dockerHubUser}/notes-app:latest"
          sh "docker push ${env.dockerHubUser}/notes-app:latest
        }

    }

```



# If i change in github jenkins automatically run.  GitHub Webhook

Step 1. Go to GitHub Repo setting. Insights ke bagal wali setting. Not GitHub account setting.

Step 2. payload URL : http://IPpubadressofec2:8080/github-webhook/   (github-webhook add this /github-webhook/)
        content type: application/x-www-from-urlencoded,
        SSL Verification : Disable (not recommended)
        Send me everything (select) , Active, create webhook

Step 3. Go to jenkins, Go to configuration,  In Build Trigger check the box of GutHub hook trigger for GITScm polling.







