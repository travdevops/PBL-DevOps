# ANSIBLE PROJECT
# Ansible Dynamic Assignments (Include) and Community Roles
Ansible is an actively developing software project, therefore you are recommended highly to always visit [Ansible Documentation](https://docs.ansible.com/) for the latest updates on modules and their usage.

We can perform configurations using **playbooks**, **roles** and **imports** from the previous Ansible Projects. 

In this project we will introduce `dynamic-assignments` by using **include** module.

### Difference between static and dynamic assignments?
- `static-assignments` uses _import_ Ansible module while `dynamic-assignments` uses the _include_ module.

Hence

    import = Static
    include = Dynamic
### IMPORT MODULE
- When you use the import module, all the statements in the playbooks are processed before they are read.
- Also means when you execute `site.yml` playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements.
- During actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

### INCLUDE MODULE
- When the include module is used, all the statements in the playbook are processed during playbook execution.
- This means that any changes to the statements encountered during execution will be taken into account.

**NB:** In most cases, it is recommended to use `static-assignments` for playbooks, because it is more **reliable**. 

With `dynamic-assignments`, it is hard to debug playbook problems due to its dynamic nature. However, you can use `dynamic-assignments` for environment specific variables as we will be introducing in this project.

### INTRODUCING DYNAMIC ASSIGNMENT INTO OUR STRUCTURE
In your `https://github.com/<your-name>/ansible-config-mgt` GitHub repository start a new branch and call it `dynamic-assignments`.


In My case here,  `https://github.com/travdevops/ansible-config-mgt`

<img width="338" alt="create-dyn-assig-branch" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/3254e57c-4531-441c-8dcd-ccba86fb5c29">


Your Github would have the following folder structure

<img width="321" alt="tree-dyn-ass" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/14e57a4a-99cc-4e38-a6df-00cd31b23497">


Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment.

For this reason, we will now create a folder to keep each environment's variables file. Therefore, 
- create a new folder `env-vars`, then for each environment, create new `YAML` files which we will use to set variables.

The folder layout becomes:

<img width="308" alt="tree-dyn-ass2" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/7f4e8538-c13a-490f-9829-51359fa90389">

Paste the code instructions below into the `env-vars.yml` file.


<img width="743" alt="env-varsyml" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/704745a8-0c44-49cd-a051-f2747e32069d">

### Notice 3 things here:

1. We used `include_vars` syntax instead of `include`, this is because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the `include` module is deprecated and variants of `include_*` must be used. These are:


- include_role
- include_tasks
- include_vars

In the same version, variants of `import` were also introduces, such as:

- import_role
- import_tasks


2. We made use of a special variables `{{ playbook_dir }}` and `{{ inventory_file }}`. `{{ playbook_dir }}` will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. `{{ inventory_file }}` on the other hand will dynamically resolve to the name of the inventory file being used, then append `.yml` so that it picks up the required file within the `env-vars` folder.


3. We are including the variables using a loop. `with_first_found` implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

### UPDATE SITE.YML WITH DYNAMIC-ASSIGNMENTS
- Update site.yml file to make use of the dynamic assignment.

<img width="746" alt="upd-siteyml-dyn" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/24f0cfec-d41e-4bf2-b1f9-7acdc1f88d34">


### Community Roles
Now it is time to create a role for MySQL database - it should install the MySQL package, create a database and configure users. Ansible already has many roles that have been developed by other open source engineers out there. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. 

Ansible Galaxy is what we will use to download a new role from the community.

### DOWNLOAD MYSQL ANSIBLE ROLE

You can browse available community roles [here](https://galaxy.ansible.com/home)

We will be using a MySQL role developed by [geerlingguy](https://galaxy.ansible.com/geerlingguy/mysql)

**Hint:** To preserve your your GitHub in actual state after you install a new role - make a commit and push to master your 'ansible-config-mgt' directory. Of course you must have git installed and configured on Jenkins-Ansible server and, for more convenient work with codes, you can configure Visual Studio Code to work with this directory. In this case, you will no longer need webhook and Jenkins jobs to update your codes on Jenkins-Ansible server, so you can disable it - we will be using Jenkins later for a better purpose.

- On our `Jenkins-Ansible server`, Make sure git is installed, if not, Install Git and Initialise Git in the `ansible-config-mgt` directory.
- Merge the changes made on Github and make sure the main branch has the latest codes and changes.
- From the Jenkins-Ansible server, Pull down the latest changes to the remote server.

If this is a fresh directory. Run these commands and get the latest from the github REPO

    git init
    git pull https://github.com/<your-name>/ansible-config-mgt.git
    git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
    git branch roles-feature
    git switch roles-feature

- Inside `roles` directory create your new MySQL role with `ansible-galaxy install geerlingguy.mysql`

<img width="1030" alt="ans-gala-geer-mysql" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/2e75f42a-4f94-4900-bbe0-968eed6283ae">

**Note that.** The file `geerlingguy.mysql` downloaded successfully but extracted to `.ansible` directory. That happened because the `.ansible` is the default dorectory for ansible roles. To avoid that and for the file to extract in the current working directory. Parse in a `-p` flag in the command. 

- Move the file back to our current working directory `~/ansible-config-mgt/roles`

      mv ~/.ansible/roles/geerlingguy.mysql ~/ansible-config-mgt/roles

- Rename the folder to `mysql`

      mv geerlingguy.mysql/ mysql

- Read `README.md` file, and edit roles configuration to use correct credentials for MySQL required for the tooling website.

This can be done on the server but it would be easier handling this 

- In the `roles/mysql/defaults/main.yml` Make some configurations

<img width="731" alt="mysql-mainyml" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/1c9d0910-8301-46ba-98dc-8c98230f4a32">

- In the `roles/mysql/tasks/main.yml`, Remove every line that has `ansible.builtin.`

<img width="1274" alt="replacing-asnbuiltin-nothing" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/ef16c393-c309-459c-b83e-8922cf5ce081">

<img width="1268" alt="replacing-asnbuiltin-nothing-result" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/79514c77-2d47-463a-915e-aa0bd87841b8">

- In the `roles/mysql/tasks/variables.yml`, Configured the 1st variable configuration play.
- Remove the `ansible.builtin.` from the code.

<img width="1143" alt="removing-ans-builtin" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/06770a37-fbe4-4ca6-8870-bed095d76d8e">

- In the `roles/mysql/tasks/configure.yml`, edit ALL Plays with `ansible.builtin.command` to just `command`

- Do the same previous step for `roles/mysql/tasks/secure-installation.yml`

- Create a `db.yml` inside `static-assignments` folder and add a play


<img width="262" alt="dbyml" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/b6d13764-e38c-49b0-8ed2-4aa58e8c7ae8">

- Update `site.yml` with importing the `db.yml` play

<img width="593" alt="siteyml-1" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/8a0db2df-a46e-471c-8235-da2f865bd2eb">


Now, The Mysql configurations ae complete. 

### LOAD BALANCER ROLES

We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively:

1. Nginx

2. Apache

- Download the Loadbalancer roles with Ansible-Galaxy

      ansible-galaxy install geerlingguy.nginx
      ansible-galaxy install geerlingguy.apache


- Move the files back to our roles directory
- Rename the files to nginx & apache respectively

<img width="706" alt="mv nginx apache" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/c9094cdc-72a1-4d76-806b-cf23cd44f523">


With our experience on Ansible so far we can:

### Important Hints:
- Since you cannot use both [Nginx](url) and [Apache](url) load balancer, you need to add a condition to enable either one - this is where you can make use of variables.
- Declare a variable in `defaults/main.yml` file inside the [Nginx](url) and [Apache](url) roles. Name each variables `enable_nginx_lb` and `enable_apache_lb` respectively.
- Configure the nginx upstreams by remove the code block and update it with the <Private-Ip-Address> of both our servers.
- Configure the `apache/defaults/main.yml` by mapping out the IP Hostname pointing to the <Private-Ip-Address>.
- Set both values to false like this `enable_nginx_lb: false` and `enable_apache_lb: false`.
- Declare another variable in both roles `load_balancer_is_required` and set its value to `false` as well
- Create `loadbalancer.yml` inside `static-assignments` and update it with this code, add `become: true` to enable superuser privileges.

<img width="676" alt="cat loadbalanceryml" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/1c84ab71-c4c1-450b-9001-cac62370f70a">

- Update both assignment and `site.yml` files respectively

  <img width="780" alt="siteyml-loadb" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/c5225236-37bd-4619-807c-47b928dfc480">


### Nginx-Config

<img width="796" alt="nginxmainyml" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/a6d03e74-ca62-4a94-81ff-a45d231eb995">



### Apache-Config


<img width="723" alt="apachemainyml" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/023b9d99-7d4a-46a2-b431-08dffec33d22">

### Setting Up The Playbook 
Now we can make use of `env-vars/uat.yml` file to define which loadbalancer to use in UAT environment by setting respective environmental variable to true.

- We will run the playbook on the UAT Env by using both Loabalancer variables set to true on [Nginx](url) and [Apache](url)

### Environment Variable for Nginx set to true:

<img width="505" alt="env-vars-uatyml-nginx" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/13cde71b-0b4f-4a02-bdc4-8dc4d0fd88fa">

**Side note:** To make our playbook run with reduced bulkiness and less default error output messages, add some configurations to the `ansible.cfg` file

    deprecation_warnings = False
    gather_facts = smart
These two configurations would remove the deprecation warnings and provide smart for the gathering facts play.

<img width="499" alt="ans-config" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/bf1e9b05-99fa-4bf1-8e82-5df0e8f2d5fc">

Note: 
- Make sure you update your `lb` and `db` servers with the inventory environment where playbook is to be run
- Make sure ssh connection has been eastablished from the Jenkins-Ansible to the webservers (`lb` & `db`)

<img width="602" alt="inv-uatyml" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/62ac6ea7-caff-44ea-b2c8-267091fef096">

- **Before Running the playbook**

You can check if our playbook codes has any error by using the playbook run command with `--syntax-check`

<img width="905" alt="playbook-syntaxcheck" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/6b6dea18-def4-49ae-89bc-a811f3d2ac9f">

If all is succesful with no errors. Run the playbook.

### Running the Playbook with Nginx LB

<img width="1382" alt="playbook1-nginx" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/29dd74d0-2802-4e8a-9f62-1f7c4b8e6d6d">

<img width="1368" alt="playbook2-nginx" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/a6cfad81-8884-49c4-becb-d6145e89362b">


### Environment Variable for Apache set to true:

- Edit the `env-vars/uat.yml` file changing the lb to apache and set to true

<img width="347" alt="env-vars-uatyml-apache" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/24b707b4-adf7-4dc9-857d-b162098c5757">


### Running the Playbook with Apache LB



<img width="1050" alt="playbook-1-apache" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/70764c00-33c0-451c-bb02-c337941d35c3">

<img width="947" alt="playbook-2apache" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/3f56e2fc-bc61-4af9-8ad2-caf23b9d0549">






<img width="570" alt="Screenshot 2023-11-15 at 10 28 36 AM" src="https://github.com/travdevops/PBL-DevOps/assets/137777644/6d9f53f8-bec4-40da-8dcd-df9ef233ef02">




