# Ansible-Refactoring-and-Static-assignments (Imports and Roles)

Before we begin, Let us make some changes to our Jenkins job - now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. Let us enhance it by introducing a new Jenkins project/job - we will require `Copy Artifact plugin`.

## Step 1 - Jenkins Job Enhancement

1. Go to Jenkins-Ansible server and create a new directory called ansible-config-artifact - we will store there all artifacts after each build.

```
sudo mkdir ansible-config-artifact
```

2. Change permissions to this directory, so Jenkins could save files there -
```
chmod -R 0777 /home/ubuntu/ansible-config-artifact
```
![image 1](https://github.com/user-attachments/assets/84ea6784-889e-4f1d-8b3c-b0f7d17470be)

3. Go to `Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab` search for Copy Artifact and install this plugin without restarting Jenkins.

![image 2](https://github.com/user-attachments/assets/504a7899-1fb5-492a-bb60-5a062f1f778a)

![image](https://github.com/user-attachments/assets/e5164324-d7c8-46aa-ae88-3f85ddf5d14a)

4. Create a new Freestyle project and name it save_artifacts. This project will be triggered by completion of your existing ansible project. Configure it accordingly:

![image](https://github.com/user-attachments/assets/52479ef1-eb19-40e2-a653-c56dc5a2fc57)

![image](https://github.com/user-attachments/assets/25869efd-c1d1-412a-9da6-b0d3281fa969)

![image](https://github.com/user-attachments/assets/52112dc7-bea9-40e5-85ed-7f257d9cb317)

![image](https://github.com/user-attachments/assets/b54aa3cf-2cff-42cc-8c96-bc97cee42345)

![image](https://github.com/user-attachments/assets/8644bf79-27a9-4885-9dbc-8ba9234d1d35)

![image](https://github.com/user-attachments/assets/dcd07642-c64c-43b7-94ac-bf2d6a77121c)


### Note: You can configure number of builds to keep in order to save space on the server, for example, you might want to keep only last 2 or 5 build results. You can also make this change to your ansible job.

The main idea of save_artifacts project is to save artifacts into /home/ubuntu/ansible-config-artifact directory. 
To achieve this, create a Build step and choose Copy artifacts from other project, specify ansible as a source project and /home/ubuntu/ansible-config-artifact as a target directory.
image

if you get an error Run `sudo chmod 755 /home/ubuntu` on your Jenkins server to give you the necessary permissioins needed.

![WhatsApp Image 2024-11-03 at 15 56 15_5861345d](https://github.com/user-attachments/assets/ea5a3314-ad43-40f5-8fe9-d5cba68e3702)

Now do a little change in your README.md file from your `ansible-config -mgt`

![WhatsApp Image 2024-11-03 at 16 01 48_a1e611fb](https://github.com/user-attachments/assets/7e6312d4-d142-439c-a25a-643adf0fc436)

It should work now!

If both Jenkins jobs have completed one after another – you shall see your files inside /home/ubuntu/ansible-config-artifact directory and it will be updated with every commit to your master branch.

Now your Jenkins pipeline is more neat and clean.


## Step 2 - Refactor Ansible code by importing other playbooks into `site.yml`

1. Setting Up for Refactoring. Ensure you have pulled the latest code from the master (main) branch. This is best practice and to ensure your project is always updated with the latest code.

```
git pull origin <branch>
```

2. Create a new branch and name it refactor

```
git branch refactor
```

3. Select the created refactor branch.
```
git checkout refactor
```

![image](https://github.com/user-attachments/assets/4b140d63-bc89-4c96-8912-7f778b0b50e9)

DevOps philosophy implies constant iterative improvement for better efficiency – refactoring is one of the techniques that can be used, but you always have an answer to question "why?". Why do we need to change something if it works well?

In Project 11 you wrote all tasks in a single playbook common.yml, now it is pretty simple set of instructions for only 2 types of OS, but imagine you have many more tasks and you need to apply this playbook to other servers with different requirements. In this case, you will have to read through the whole playbook to check if all tasks written there are applicable and is there anything that you need to add for certain server/OS families. Very fast it will become a tedious exercise and your playbook will become messy with many commented parts. Your DevOps colleagues will not appreciate such organization of your codes and it will be difficult for them to use your playbook.

Most Ansible users learn the one-file approach first. However, breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

Let see code re-use in action by importing other playbooks.

1. Within playbooks folder, create a new file and name it `site.yml`. This file will now be considered as an entry point into the entire infrastructure configuration.
![image](https://github.com/user-attachments/assets/b2f68f39-7857-46e1-b86b-d208527f369e)

Other playbooks will be included here as a reference. In other words, `site.yml` will become a parent to all other playbooks that will be developed. Including common.yml that you created previously. Dont worry, you will understand more what this means shortly.

2. Create a new folder in root of the repository and name it `static-assignments`. The `static-assignments` folder is where all other children playbooks will be stored. This is merely for easy organization of your work. It is not an Ansible specific concept, therefore you can choose how you want to organize your work. You will see why the folder name has a prefix of static very soon. For now, just follow along.

```
cd ..
mkdir static-assignments
```

![image](https://github.com/user-attachments/assets/f5862b42-b9c5-447d-a52c-ce3406b28936)

3. Move `common.yml` file into the newly created `static-assignments` folder.

```
mv playbooks/common.yml static-assignments
```
After this, the `common.yml` file will reside within `static-assignments`, separating it from the main entry point (site.yml) in playbooks.

![image](https://github.com/user-attachments/assets/95dcd0e4-36e4-4923-a3c7-77625a5105b5)

4. Import the common.yml Playbook into `site.yml`:
In `site.yml`, add an import statement to include the `common.yml` playbook. This modular approach lets you expand `site.yml` to include additional playbooks as your infrastructure grows.

Update the site.yml file:

```
---
- hosts: all
  - import_playbook: ../static-assignments/common.yml
```

5. Verify Folder Structure
After completing the reorganization, your project folder structure should look like this:

```
├── static-assignments
│   └── common.yml
├── inventory
│   ├── dev
│   ├── stage
│   ├── uat
│   └── prod
└── playbooks
    └── site.yml
```

6. Create a New Playbook for Package Deletion: Next, you’ll create a separate playbook for deleting the Wireshark utility from specific servers. Within the static-assignments folder, create a new playbook named common-del.yml to delete Wireshark

```
---
- name: Update web, nfs servers
  hosts: webservers, nfs
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: Delete wireshark
      yum:
        name: wireshark
        state: removed

- name: Update LB server and db server
  hosts: lb, db
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Delete wireshark
      apt:
        name: wireshark
        state: absent
        autoremove: yes
        purge: yes
        autoclean: yes

```
![image](https://github.com/user-attachments/assets/338be53f-9f1e-4f25-9216-53aa90ab823f)

### Explanation: This playbook targets different hosts (webservers, nfs, db, and lb) to delete Wireshark. It uses yum for Red Hat-based systems and apt for Ubuntu-based systems.

7. Update site.yml to Include the New Playbook: After creating common-del.yml, update site.yml to reference it instead of common.yml.
Update site.yml to include the new playbook:

```
---
- hosts: all
  - import_playbook: ../static-assignments/common-del.yml 
```

8. Execute the Playbook in the dev Environment: To run the playbook against your dev environment, Navigate to the project directory:

```
cd /home/ubuntu/ansible-config-mgt/
```
9. Run the ansible-playbook command targeting the dev environment:

```
ansible-playbook -i inventory/dev.yml playbooks/site.yml
```

![image](https://github.com/user-attachments/assets/b706b03a-8f0b-499a-bea5-490264f753df)

10. After the playbook completes, verify Wireshark has been deleted from all servers by running:

```
wireshark --version
```
If Wireshark is removed, this command should indicate it is not installed.

![image](https://github.com/user-attachments/assets/aa8b9e93-db46-4163-b0c3-c23c3d3a474b)

Refactoring with import_playbook helps organize complex Ansible configurations, making them easier to understand, extend, and manage across different server environments. The final site.yml acts as the single command point to orchestrate your infrastructure tasks efficiently.

## Step 3 - Configuring UAT Webservers with a Role 'Webserver'
To efficiently configure your UAT web servers, we’ll be using an Ansible role to organize tasks. This step-by-step guide will help you set up, configure, and manage two new RHEL 8 EC2 instances as UAT web servers.

1. Launch UAT Web Server Instances:
   
* Launch two fresh RHEL 8 EC2 instances on AWS, which will serve as your UAT web servers. Name them:
    * Web1-UAT
    * Web2-UAT
    Tip: Stop any other EC2 instances that aren’t currently needed to avoid unnecessary costs. For this step, you only need:
        * Two RHEL 8 servers (UAT web servers)
        * One Jenkins-Ansible server (already running for Ansible automation)

2. Create the Role Structure
* In the ansible-config-mgt directory, create a roles directory:
```
mkdir roles
cd roles
```
![image](https://github.com/user-attachments/assets/daf9e32d-2671-4158-bfaf-46295ae676c2)

Alternatively, you can create the directory and files manually. The folder structure should resemble this:

```
└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    └── templates
```

Note: Remove unnecessary directories `(files, tests, and vars)` to keep things organized.

3. Update the UAT Inventory
Go to the inventory directory in ansible-config-mgt and update uat.yml with the IP addresses of your UAT servers. Ensure you add your private IP addresses:

```
[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'
```
![image](https://github.com/user-attachments/assets/e8581c56-0783-4fc8-9f33-26f7e18b2b0e)

4. In `/etc/ansible/ansible.cfg` file uncomment `roles_path` string and provide a full path to your roles directory. If you can’t locate the /etc/ansible/ansible.cfg file, there are a couple of steps you can try to locate or create it:

5. Configure SSH Agent: Ensure you are using ssh-agent for SSH access to your Jenkins-Ansible server.
   
![image](https://github.com/user-attachments/assets/9f4c9556-f0cf-4621-8790-ec35f83e4870)

   
6. Check the default Ansible configuration file: Run the following command in the terminal to confirm the location of the configuration file Ansible is using:

```
ansible --version
```
This will show you the default path for the `ansible.cfg` file, which is usually `/etc/ansible/ansible.cfg`. If it’s missing, it may need to be created.

7. Create the configuration file: If the file doesn’t exist in /etc/ansible, you can create it by running:

```
sudo mkdir -p /etc/ansible
sudo touch /etc/ansible/ansible.cfg
```

8. Then, open it to add or edit the roles_path setting:
   
```
sudo nano /etc/ansible/ansible.cfg
```
![image](https://github.com/user-attachments/assets/ff102390-4f08-42a2-a323-4a519cf330e1)

Save and close the file: Press CTRL + X to exit Nano, then Y to confirm saving, and Enter to save the file under the same name.

9. In the configuration file, look for the roles_path line (or add it if it’s missing), and specify the path:

```
roles_path = /home/ubuntu/ansible-config-mgt/roles
```
![image](https://github.com/user-attachments/assets/44734a4c-6984-4d8e-8dd3-9d4d1e6c2f4f)

10. Steps to Add Logic to the Webserver Role. Navigate to the tasks directory in your webserver role:

```
cd ~/ansible-config-mgt/roles/webserver/tasks
```

11. Edit or Create main.yml: If the file doesn't exist, create it:

```
touch main.yml
```

12. Open the file using your preferred editor:

```
nano main.yml
```

Add the Following Tasks to main.yml:

```
---
- name: Install Apache
  become: true
  ansible.builtin.yum:
    name: httpd
    state: present

- name: Install Git
  become: true
  ansible.builtin.yum:
    name: git
    state: present

- name: Clone the Tooling repository
  become: true
  ansible.builtin.git:
    repo: https://github.com/<your-name>/tooling.git
    dest: /var/www/html
    force: yes

- name: Move content to parent directory
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start httpd service
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: Remove redundant html directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```

Replace Placeholder `(<your-name>):` Replace `<your-name>` with your GitHub username in the repository URL.

## Step 3 - Create UAT Webservers Assignment:

1. Create static-assignments/uat-webservers.yml:

```
---
- hosts: uat-webservers
  roles:
     - webserver
```

2. Update playbooks/site.yml to include the new assignment:

```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml
```

3. Commit your changes by running this command:

```
git add . && git commit -m "Added webserver role logic and updated configuration" && git pull origin main --rebase && git push origin main
```

![image](https://github.com/user-attachments/assets/f1a2b2e3-3d0d-493d-8cc4-dbfc780f4000)

4. Create Pull Request:

![image](https://github.com/user-attachments/assets/f9d0927b-50c1-4565-88ab-57bcca3a6260)

5. Ensure that the webhook triggers two consecutive Jenkins jobs and that they run successfully.

![image](https://github.com/user-attachments/assets/269d2be8-0be0-493f-b86a-5d1f799cc886)

6. Execute Playbook. Run the playbook against the UAT inventory:

![image](https://github.com/user-attachments/assets/e7bd2541-6dcd-4879-a381-80bfd8f78bf2)

### Also do not forget to update your .ssh/config file adding the two new web servers created.
![image](https://github.com/user-attachments/assets/e37dfec3-3c46-4a4a-9984-f268edd47570)



7. Verify Deployment. Access the UAT webservers through a web browser:

```
http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php
http://<Web2-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php
```

![image](https://github.com/user-attachments/assets/a21b756b-c74a-4b63-a5dd-87f5f1d176bd)

### Web1-UAT-Server

![image](https://github.com/user-attachments/assets/b414d55f-6964-494c-b78d-997c19eae9f1)

### Web2-UAT-Server


## CONCLUSION
With this, we come to the end of the Ansible refactoring and implementation of static assignments/project. This new structure we implemented improves code organization, reusability, and maintainability. The Ansible architecture now includes dedicated roles and a more modular approach to configuration management

This is what your new architecture should look like now.
![image](https://github.com/user-attachments/assets/dce11028-b6ac-48ed-a817-f9295cf96d1f)















