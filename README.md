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













































