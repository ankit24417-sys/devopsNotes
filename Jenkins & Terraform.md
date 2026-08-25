# Jenkins

it is an open source software
it helps developers automate automate task like building , testing , deploying coding
it mainly used in devops for CI/CD

# Installation of Jenkins

To use jenkins , you have java on your system

> setup gpg key of jenkins
> use command => sudo apt install jenkins -y
> after installation start and enable the jenkins using the commands
> sudo systemctl start jenkins
> sudo systemctl enable jenkins

# Accessing jenkins

> run in your browser => http://<your public ip>:8080
> it will ask you the initialAdminPassword
> sudo cat /var/lib/jenkins/secrets/initialAdminPassword
> then set your credentials

# Jenkins Jobs

> A jenkins job is a simple task will jenkins run
> This task can be: build code , test it, run scripts, deploy an app etc
> Think of it as a to-do task for jenkins like "Run these steps in order"

# Jenkins workspace

> Every jenkins job has its own workspace = a folder on your system
> it stores temporary files, source code, build outputs
> You can view it by clicking : Job => Workspace
> Build history shows all past runs (successfull ans failed builds)
> You can click on any build to see its output , logs, artifacts

# Creating a job

> To create a job jenkins , you will show create job option , click on it
> you will see many options , select one of them
> and proceed

> To give localhost a global url , we use the tool ngrok

# Setting up Jenkins Build using Github

> Dashboard - New Item - Enter Job name - Pipeline (or free style ) - ok
> Connect Github => repository url - add credentials (if private , username, personal access token)
> Enable Github webhook trigger
> Create a jenkins file

# Jenkins file

> Jenkins is text file which contains pipeline as code
> It defines step like build, test, deploy in script format
> It usually saved in the root of your project repo
> Makes your CI/CD pipeline version-controlled along with the code

# Types of Pipeline in Jenkins

(i) Declarative (easy)
(ii) Scripted (Groovy based)

# Declarative jenkins file structure

```pipeline{
     agent any
       environments{
        # write you env here
       }
       stages{
         stage('name of stage'){
             step{
                 # write steps here
             }
         }
         stage('name of stage'){
             step{
                 # write steps here
             }
         }
       }
}
```

# Scripted jenkins file structure

```node {
     stage('name of stage'){
        # can add steps (optional)
        # write code here
        # you can also use try catch block
     }
     stage('name of stage'){
       # can add steps (optional)
        # write code here
        # you can also use try catch block
     }
     stage('name of stage'){
        # can add steps (optional)
        # write code here
        # you can also use try catch block
     }
}
```

[note] To create variable in jenkins file we use the keyword def

> node{
> def app='/var/www/next-js'
> }
