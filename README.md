# 100 Days of DevOps with KodeKloud - Days 81 - 90

We finished up Jenkins and then we'll be focused on Ansible for a while! Almost at the finish line. Lets get it!

## Day 90: Managing ACLs Using Ansible
## Day 89: Ansible Manage Services
## Day 88: Ansible Blockinfile Module
## Day 87: Ansible Install Package
## Day 86: Ansible Ping Module Usage
## Day 85: Create Files on App Servers using Ansible
## Day 84: Copy Data to App Servers using Ansible
## Day 83: Troubleshoot and Create Ansible Playbook
## Day 82: Create Ansible Inventory for App Server Testing
## Day 81: Jenkins Multistage Pipeline

This was even easier than the last lab. I didn't really need AI for this besides a syntax error I kept running into with my pipeline (there was some open bracket somewhere). The last thing AI did was full-proof my curl test byt adding the `-fsS` flags to show if the curl test forked or failed. That's new to me and I don't mind using AI for that since I'm not a web developer. Other than that, I took my code from previous projects that used mutli-stage builds and edited it. Don't forget to run the Java upgrade command if your Agent node is giving you issues coming online. 

THAT WRAPS UP JENKINS!!! Wow we're learning so much. We're almost there. 

```
pipeline {
    agent {
        label 'stapp01'
    }

    stages {
        stage('Deploy') {
            steps {
                sh '''
                    cd /var/www/html/
                    git checkout master
                    git pull origin master
                '''
            }
        }
        stage('Test') {
            steps {
                sh '''
                    curl -fsS http://stlb01:8091
                '''
            }
        }
    } 
}
```
