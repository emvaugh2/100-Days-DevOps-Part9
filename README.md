# 100 Days of DevOps with KodeKloud - Days 81 - 90

We finished up Jenkins and then we'll be focused on Ansible for a while! Almost at the finish line. Lets get it!

## Day 90: Managing ACLs Using Ansible
## Day 89: Ansible Manage Services
## Day 88: Ansible Blockinfile Module
## Day 87: Ansible Install Package
## Day 86: Ansible Ping Module Usage
## Day 85: Create Files on App Servers using Ansible

This lab was pretty easy too except on particular piece. We needed to create an empty file on all 3 app servers which we previously did. The new part was configuring this file with permissions AND automatically creating the right user and group. Once again, I consulted with the Ansible docs to see the format for the permissions and ownership under the ansible.builtin.file guide. Everything was good except the ownership. We needed the playbook to automatically assign the ownership based on the app server it was on. So tony for stapp01, steve for stapp02, and banner for stapp03. I had no clue how to do this. AI gave me a complicated answer the first time syntax-wise so I asked for a more simple solution. It told me to put the file owner in the inventory file. Then, it gave me the syntax for the owner and group fields in the playbook file. 

```
[app_servers]
stapp01 ansible_user=tony ansible_password=Ir0nM@n ansible_become_password=Ir0nM@n file_owner=tony
stapp02 ansible_user=steve ansible_password=Am3ric@ ansible_become_password=Am3ric@ file_owner=steve
stapp03 ansible_user=banner ansible_password=BigGr33n ansible_become_password=BigGr33n file_owner=banner
```

```
---
- name: Create an empty file on App Server 3
  hosts: app_servers
  become: true

  tasks:
  - name: The job
    ansible.builtin.file:
      path: /opt/nfsdata.txt
      state: touch
      mode: '0644'
      owner: "{{ file_owner }}"
      group: "{{ file_owner }}"
```

I ran a ping command to test connectivity. Then, I ran the playbook and verified the files were there with the proper permissions and ownership. 

## Day 84: Copy Data to App Servers using Ansible

The goal of this lab was to create an Ansible playbook that copies file from the jumpbox host to all of the application servers. I created an inventory file from my previous lab with all the three application servers:

```
[app_servers]
stapp01 ansible_user=tony ansible_password=Ir0nM@n ansible_become_password=Ir0nM@n
stapp02 ansible_user=steve ansible_password=Am3ric@ ansible_become_password=Am3ric@
stapp03 ansible_user=banner ansible_password=BigGr33n ansible_become_password=BigGr33n
```

Then, I tested connectivity to the servers using `ansible all -i inventory -m ping`. Then, I created my runbook. Now, I went to the Ansible docs so I can try to create this from scratch. The documentation isn't too bad. I found the `ansible.builtin.copy` command on the site and used the examples to come up with this:

```
---
- name: Copy file from jumphost to all 3 application servers
  hosts: app_servers
  become: true

  tasks:
  - name: scp for App Servers 
    ansible.builtin.copy:
      src: /usr/src/finance/index.html
      dest: /opt/finance
```

Now, at first, I created a task for all 3 server until AI reminded me that this playbook will run against all three servers one by one. The inventory file has the credentials as well so I didn't need to state the user and hostname for the remote server. Once I erased that part, my playbook ran smoothly and I verified that the `index.html` file was present on the three machines. Easy peasy. 

## Day 83: Troubleshoot and Create Ansible Playbook

This was prety straightforward as well. We needed to fix the inventory file and then create a playbook to create an empty file on `stapp03`. In the inventory file, I immediately saw it was missing the `[<host_group>]` line at the very top. The rest of the file looked okay so once I changed that, I did a ping test using `ansible all -i <inventory_file_name> -m ping`. That failed so I looked at the inventory file again. The password variable looked wrong to me so I just passed the actual password to the file. The ping command worked after that. 

I created the playbook file from another file I had. AI needed to edit it. I've never once in my life saw the `ansible.builtin.file` command. Yes, I've seen the `ansible.builtin` part before but never the file extension. Still a pretty simple YAML file though. Once I did that, I ran the playbook using the `ansible-playbook -i <inventory_file_name> <playbook_file_name>` command. Lab completed!

```
---
- name: Create an empty file on App Server 3
  hosts: app_servers
  become: true

  tasks:
  - name: The job
    ansible.builtin.file:
      path: /tmp/file.txt
      state: touch
```

## Day 82: Create Ansible Inventory for App Server Testing

Lets go over the basics. Ansible is used to configure your actual machine, not the environment. Say you wanted to deploy an entire cloud environment. You would use Terraform. But what about the actual VMs in that environment? How should they be configured? You can use Ansible to automate this. 

For this lab, we needed to create an inventory file and then run the corresponding playbook. The playbook was already created. We were instructed to put `stapp02` into the inventory file. Very simple. I'm used to doing this with IP addresses and SSH keys so that was the main difference here. I put the hostname and the username but I didn't know about the password part. AI helped me with this. 

Regarding the `ansible_become_password` phrase, I believe the become password is the root password. In the playbook, it has `become:` which tells Ansible to user a specific account (I think the default is root). I THINK the next line `become_user:` says "become THIS user". 

```
[app_servers]
stapp02 ansible_user=steve ansible_password=Am3ric@ ansible_become_password=Am3ric@
```

This was pretty easy. Lets move to the next one. 

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
