## Documentation of Project 12 
---

**ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)**
-
In this project, I worked with ansible-config-mgt repository and made some improvements on my code. I refactored my Ansible code, created assignments(directory), and learnt how to use the imports functionality which allowed me to effectively re-use previously created playbooks in a new playbook effortlessly.

*Step 1 – Jenkins job enhancement*
=
We need to make some changes to our Jenkins job but remember that every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. The main purpose of DevOps is mainly about improvement, automating processes, and effecting chnages to make programming easier, effective and faster. We will be introducing a new Jenkins project/job – we will require Copy Artifact plugin.
- **Follow the steps below to achieve this task:**
  - Go to your Jenkins-Ansible server and create a new directory called ansible-config-artifact – we will store there all artifacts after each build. Create a new directory on your Jenkins terminal with `sudo mkdir /home/ubuntu/ansible-config-artifact`
  - Change permissions to this directory, so Jenkins could save files there – chmod -R 0777 /home/ubuntu/ansible-config-artifact
  - Go to Jenkins web console **-> Manage Jenkins** **-> Manage Plugins** > on **Available** tab search for **Copy Artifact and install this plugin without restarting Jenkins**
  - Create a new **Freestyle project** like we did in Project 9 and name it `save_artifacts`
  - This project will be triggered by completion of your existing ansible project. Configure it accordingly. See the image below: ![Jenkins Configuration](./images/Jenkins%20Configuration.PNG)
  - The main idea of `save_artifacts` project is to save artifacts into `/home/ubuntu/ansible-config-artifact` directory a utomatically. To achieve this, you will need to create a **Build step** and choose **Copy artifacts** from other project, **specify ansible as a source project** and `/home/ubuntu/ansible-config-artifact` as a target directory. See the image below: ![Jenkins Setup](./images/Jenkins%20setup.PNG)
  - You can test your set up by making some change in README.MD file inside your ansible-config-mgt repository (right inside main branch). If both Jenkins jobs have completed one after another, you should see your files inside `/home/ubuntu/ansible-config-artifact` directory and it will be updated with every commit to your main branch.
- Now let's refactor ansible code by importing other playbooks into `site.yml` However, before starting to refactor the codes, ensure that you have pulled down the latest code from (main)branch, and created a new branch, name it refactor. Rember I mentioned earlier that DevOps philosophy implies constant iterative improvement for better efficiency and in this present case **refactoring** is one of the techniques that can be used but you might be wondering why we have to do that when the previous ones worked well enough. Try and imagine you have many more tasks and you need to apply this playbook to other servers with different requirements. In this case, you will have to read through the whole playbook to check if all tasks written there are applicable and is there anything that you need to add for certain server/OS families. Very fast it will become a tedious exercise and your playbook will become messy with many commented parts. Your DevOps colleagues will not appreciate such organization of your codes and it will be difficult for them to use your playbook. Follow the steps below to know how to skip the tedious procedures.
- *Within playbooks folder, create a new file and name it `site.yml` - This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, `site.yml` will become a parent to all other playbooks that will be developed. Including common.yml that you created previously.*
- Create a new folder in root of the repository and name it `static-assignments`. The `static-assignments` folder is where all other children playbooks will be stored for easy organisation of your work.
- Move `common.yml` file into the newly created `static-assignments` folder
- *Inside `site.yml` file, import `common.yml` playbook.* Your directory structure should like this: ![File Structure](./images/File%20structure.PNG)
- Run `ansible-playbook` command against the `dev` environment. ince you need to apply some tasks to your dev servers and wireshark is already installed – you can go ahead and create another playbook under static-assignments and name it `common-del.yml`. In this playbook, configure deletion of wireshark utility. In the `common-del.yml` file, copy and paste the following codes:
```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
      
```
- - update `site.yml` with `- import_playbook: ../static-assignments/common-del.yml` instead of `common.yml` and run it against dev server and change directory to  `/home/ubuntu/ansible-config-artifact/` and run `ansible all -m ping` to confirm the availability of the host. The image below should be your output. ![Hosts Ping](./images/ansible-playbook%20command.PNG) Now run the this command to effect the changes made. `ansible-playbook -i inventory/dev.yml playbooks/site.yml`. See the image below for the expected output- ![Wireshark deletion](./images/Deleted%20wireshark.PNG) Now check all the servers to confirm the deleted wireshark. Check the images below: ![NFS](./images/nfs%20wireshark%20deleted.PNG) ![Webserver1](./images/web1%20wireshark%20deleted.PNG) ![Webserver2](./images/web2%20wireshark%20deleted.PNG) ![Loadbalancer](./images/lb%20proof%20of%20wireshark%20deleted.PNG) ![Database](./images/database%20wireshark%20deleted.PNG)

**CONFIGURE UAT WEBSERVERS WITH A ROLE ‘WEBSERVER’**
 - Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – `Web1-UAT` and `Web2-UAT`.
 - To create a role, you must create a directory called roles/, relative to the playbook file or in `/etc/ansible/` directory
 - Create the directory/files structure manually. In the root folder, create a folder and name it `webserver`and inside webserver folder, create five more folders namely: `README.md`, `defaults`(create a file inside it and name it `main.yml`), `handlers`(inside handlers, create a file and name it `main.yml`), `meta`(jnside this folder, create a file and name it `main.yml`), `tasks`(inside tasks, create a file and name it `main.yml`). Your enire file structure should look like the image below: ![File Structure](./images/Structure%20of%20the%20workspace.PNG)
 - Update your inventory `ansible-config-mgt/inventory/uat.yml` file with IP addresses of your 2 UAT Web servers. Ensure you are using ssh-agent to ssh into the Jenkins-Ansible instance just as we did in project 11. In  your `uat.yml` file, edit and paste the following codes:
 ```
 [uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 

<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 
```
- In `/etc/ansible/ansible.cfg` file uncomment `roles_path` string and provide a full path to your roles directory `roles_path`    `= /home/ubuntu/ansible-config-mgt/roles`, so Ansible could know where to find configured roles.
- It is time to start adding some logic to the webserver role. Go into tasks directory, and within the `main.yml` file, start writing configuration tasks to do the instructions in the folllowing codes. Ensure you edit and add your github username provided you have the toolinf file on your account.
```
---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/<your-name>/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```
- **REFERENCE WEBSERVER ROLE**
---
- Within the static-assignments folder, create a new assignment for uat-webservers `uat-webservers.yml`. This is where you will reference the role.
```
---
- hosts: uat-webservers
  roles:
     - webserver
```
- Remember that the entry point to our ansible configuration is the `site.yml` file. Therefore, you need to refer your `uat-webservers.yml` role inside `site.yml.`
```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml
```
- Commit your changes, create a Pull Request and merge them to master branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into `/home/ubuntu/ansible-config-mgt/` directory.
- Now run the playbook against your `uat inventory`, `run sudo ansible-playbook -i /home/ubuntu/ansible-config-artifact/inventory/uat.yml /home/ubuntu/ansible-config-artifact/playbooks/site.yml`, the image below should be your output: ![Final output](./images/ansible-playbook%20second%20task.PNG) 
- You should be able to see both of your UAT Web servers configured and you can try to reach them from your browser using `http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php 
(for web1) and http://<Web2-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php (for web2) Do well to edit it before browsing it on the web(You should see the tooling website successfully deployed). Check the images below for my own results:
![WEB1](./images/Tooling%20website%20for%20webserver1.PNG)
![WEB2](./images/tooling%20website%20for%20webserver2.PNG)

**Congratulations on the successful completion of this Project! Thanks for your patience. Have a nice day!**
--