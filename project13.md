# ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES

Well, from Project 12, you can already tell that static assignments use import Ansible module. The module that enables dynamic assignments is <b>include</b>.

    import = Static
    include = Dynamic

When the import module is used, all statements are pre-processed at the time playbooks are parsed. Meaning, when you execute site.yml playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

On the other hand, when include module is used, all statements are processed only during execution of the playbook. Meaning, after the statements are parsed, any changes to the statements encountered during execution will be used.


## Part 1 – Introducing Dynamic Assignment Into Our structure

In your https://github.com/your-name/ansible-config-mgt GitHub repository start a new branch and call it dynamic-assignments.

Create a new folder, name it dynamic-assignments. Then inside this folder, create a new file and name it env-vars.yml. We will instruct site.yml to include this playbook later.</p>

Your GitHub shall have following structure by now:

    ├── dynamic-assignments
    │   └── env-vars.yml
    ├── inventory
    │   └── dev
        └── stage
        └── uat
        └── prod
    └── playbooks
        └── site.yml
    └── roles (optional folder)
        └──...(optional subfolders & files)
    └── static-assignments
        └── common.yml

Create a new folder env-vars, then for each environment, create new YAML files which we will use to set variables.

Your layout should now look like this.

    ├── dynamic-assignments
    │   └── env-vars.yml
    ├── env-vars
        └── dev.yml
        └── stage.yml
        └── uat.yml
        └── prod.yml
    ├── inventory
        └── dev
        └── stage
        └── uat
        └── prod
    ├── playbooks
        └── site.yml
    └── static-assignments
        └── common.yml
        └── webservers.yml
 
Now paste the instruction below into the env-vars.yml file:

    ---
    - name: collate variables from env specific file, if it exists
      hosts: all
      tasks:
        - name: looping through list of available files
          include_vars: "{{ item }}"
          with_first_found:
            - files:
                - dev.yml
                - stage.yml
                - prod.yml
                - uat.yml
              paths:
                - "{{ playbook_dir }}/../env-vars"
          tags:
            - always

Update site.yml file to make use of the dynamic assignment:

    ---
    - hosts: all
    - name: Include dynamic variables 
      tasks:
      import_playbook: ../static-assignments/common.yml 
      include: ../dynamic-assignments/env-vars.yml
      tags:
        - always

    -  hosts: webservers
    - name: Webserver assignment
      import_playbook: ../static-assignments/webservers.yml
  
To preserve our GitHub in actual state after we install a new role; make a commit and push to master your ‘ansible-config-mgt’ directory.
  
Now it is time to create a role for MySQL database – it should install the MySQL package, create a database and configure users. With Ansible Galaxy, we can simply download a ready to use ansible role, and keep going.

Download Mysql Ansible Role: We will be using a MySQL role developed by geerlingguy

On Jenkins-Ansible server make sure that git is installed with git --version, then go to ‘ansible-config-mgt’ directory and run:

    git init
    git pull https://github.com/<your-name>/ansible-config-mgt.git
    git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
    git branch roles-feature
    git switch roles-feature

Inside roles directory create your new MySQL role with:

    ansible-galaxy install geerlingguy.mysql

Rename the folder to mysql:

    mv geerlingguy.mysql/ mysql

Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website.

Now it is time to upload the changes into your GitHub:

    git add .
    git commit -m "Commit new role files into GitHub"
    git push --set-upstream origin roles-feature

We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively:

- Nginx
- Apache

Decide if you want to develop your own roles, or find available ones from the community. In this case we got our roles from Ansible Galaxy with these commands:

    ansible-galaxy install geerlingguy.apache
    ansible-galaxy install geerlingguy.nginx

Update both static-assignment and site.yml files to refer the roles

Since you cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one – this is where you can make use of variables.

Declare a variable in defaults/main.yml file inside the Nginx and Apache roles. Name each variables enable_nginx_lb and enable_apache_lb respectively.

Set both values to false like this enable_nginx_lb: false and enable_apache_lb: false.

Declare another variable in both roles load_balancer_is_required and set its value to false as well

Update both assignment and site.yml files respectively

- loadbalancers.yml file

      - hosts: lb
        roles:
          - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
          - { role: apache, when: enable_apache_lb and load_balancer_is_required }

- site.yml file

       - name: Loadbalancers assignment
         hosts: lb
           - import_playbook: ../static-assignments/loadbalancers.yml
          when: load_balancer_is_required 

Now you can make use of env-vars\uat.yml file to define which loadbalancer to use in UAT environment by setting respective environmental variable to true.

You will activate load balancer, and enable nginx by setting these in the respective environment’s env-vars file.

    enable_nginx_lb: true
    load_balancer_is_required: true

Run the Ansible Playbook:

    ansible-playbook -i inventory/uat playbooks/site.yml

### NB: Please note that Database and Load Balance instance has been changed from Ubuntu to RedHat, so the Private IPs has been affected as well

- Change nginx_user to just nginx in Ansible-config-mgt/roles/nginx/templates/nginx.conf.j2
 
- Also modified to add Upstream details in Ansible-config-mgt/roles/nginx/templates/nginx.conf.j2

  
  
