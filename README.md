# Tomcat Ansible Role:

* Ansible role to install and configure Apache Tomcat on CentOS/RHEL.

* Tomcat supported versions by this role:
  * 8.5
  * 9.0

* Integration with Jenkins pipeline.

## Prerequisities:

* Ansible package must be installed.
```
$ ansible --version
  ansible 2.9.6
  config file = /root/tomcat-ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5
```

## Tasks in the role:

* Install Java.
* Add tomcat user and group.
* Download tomcat and install.
* Configure tomcat.
* Initialize tomcat service.

## How to use this role:

**1. Clone the Project:**

```
$ git clone https://github.com/shivanikothari/tomcat-server-installation.git
$ cd tomcat-server-installation
```
**2. Update your inventory:**

```
$ vi inventory/hosts
[tomcat-nodes]
node1 ansible_ssh_host=127.0.0.1 ansible_user=root
```
**3. Default Installation:**
* This playbook will by default install one tomcat instance on each node.
* In order to add multiple installation on same server, we can override the default variables.

```
$ vi plays/roles/tomcat-installation/defaults/main.yml
...
tomcat_users:
# - [ 'user', 'group', 'password', 'home', 'shutdown_port', 'listen_address', 'connector_port', 'redirect_port', 'ajp_port' ]
  - [ 'user1', 'tomcat', 'changeme', "/home/user1", '8005', '0.0.0.0', '8080', '8443', '8009' ]
  - [ 'user2', 'tomcat', 'changeme', "/home/user2", '8006', '0.0.0.0', '8081', '8444', '8010' ]
...
```

* Also, while configuring multiple tomcat servers on same machine you will need to change below listed port:
    * shutdown_port
    * connector_port
    * redirect_port
    * ajp_port

**4. Default tomcat version:**
* By default, this playbook will install v8.5.2. 
* This can be changed by modifying the default vars or by passing --extraVars while running ansible playbook.**

**5. Running Playbook:**<br/>
Once all values are updated, you can then run the playbook against your nodes.

```
# With sudo user password:
$ ansible-playbook plays/tomcat-installation.yml --ask-become-pass

# With root user password:
$ ansible-playbook plays/tomcat-installation.yml --ask-pass

# With sudo user and password:
$ ansible-playbook plays/tomcat-installation.yml --ask-pass --ask-become-pass

# With passwordless root user:
$ ansible-playbook plays/tomcat-installation.yml 
PLAY [Tomcat Deployment] ***********************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [node1]

TASK [tomcat-installation : Install Java] ******************************************************************************************
ok: [node1]

TASK [tomcat-installation : Install Java] ******************************************************************************************
skipping: [node1]

TASK [tomcat-installation : Create tomcat users] ***********************************************************************************
changed: [node1] => (item=[u'user1', u'tomcat', u'changeme', u'/home/user1', u'8005', u'0.0.0.0', u'8080', u'8443', u'8009'])
changed: [node1] => (item=[u'user2', u'tomcat', u'changeme', u'/home/user2', u'8006', u'0.0.0.0', u'8081', u'8444', u'8010'])

TASK [tomcat-installation : Download apache-tomcat-8.5.42.tar.gz] ******************************************************************
changed: [node1]

TASK [tomcat-installation : Unarchive apache-tomcat-8.5.42.tar.gz] *****************************************************************
skipping: [node1] => (item=[u'user1', u'tomcat', u'changeme', u'/home/user1', u'8005', u'0.0.0.0', u'8080', u'8443', u'8009'])
skipping: [node1] => (item=[u'user2', u'tomcat', u'changeme', u'/home/user2', u'8006', u'0.0.0.0', u'8081', u'8444', u'8010'])

TASK [tomcat-installation : fix permissions] ***************************************************************************************
ok: [node1] => (item=[u'user1', u'tomcat', u'changeme', u'/home/user1', u'8005', u'0.0.0.0', u'8080', u'8443', u'8009'])
ok: [node1] => (item=[u'user2', u'tomcat', u'changeme', u'/home/user2', u'8006', u'0.0.0.0', u'8081', u'8444', u'8010'])

TASK [tomcat-installation : Clean up temporary files] ******************************************************************************
ok: [node1]

TASK [tomcat-installation : Setup server.xml] **************************************************************************************
ok: [node1] => (item=[u'user1', u'tomcat', u'changeme', u'/home/user1', u'8005', u'0.0.0.0', u'8080', u'8443', u'8009'])
ok: [node1] => (item=[u'user2', u'tomcat', u'changeme', u'/home/user2', u'8006', u'0.0.0.0', u'8081', u'8444', u'8010'])

TASK [tomcat-installation : Copy tomcat service file] ******************************************************************************
ok: [node1] => (item=[u'user1', u'tomcat', u'changeme', u'/home/user1', u'8005', u'0.0.0.0', u'8080', u'8443', u'8009'])
ok: [node1] => (item=[u'user2', u'tomcat', u'changeme', u'/home/user2', u'8006', u'0.0.0.0', u'8081', u'8444', u'8010'])

TASK [tomcat-installation : Start and enable tomcat service] ***********************************************************************
ok: [node1] => (item=[u'user1', u'tomcat', u'changeme', u'/home/user1', u'8005', u'0.0.0.0', u'8080', u'8443', u'8009'])
ok: [node1] => (item=[u'user2', u'tomcat', u'changeme', u'/home/user2', u'8006', u'0.0.0.0', u'8081', u'8444', u'8010'])

PLAY RECAP *************************************************************************************************************************
node1                      : ok=9    changed=2    unreachable=0    failed=0
```

**6. Verify tomcat service:**
```
$ systemctl status user1-tomcat.service
● user1-tomcat.service - Apache Tomcat Web Application
   Loaded: loaded (/etc/systemd/system/user1-tomcat.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2020-04-26 08:36:41 IST; 6h ago
 Main PID: 4482 (java)
 ......
```

## Execute Ansible playbook with Jenkins pipeline:

**1. Install Ansible plugin:**
* In order to integrate Jenkins and ansible playbooks, we need to install Ansible plugin on Jenkins.
* Steps to install plugin:
   1. Go to Manage Jenkins 
   2. Manage Plugins 
   3. Available 
   4. Search for Ansible
   ![Ansible Plugin](/images/ansible_plugin.png)
* If you have already installed the Plugin, it will be under Installed section.
  
**2. Create Pipeline Job:**
* Jenkins Pipeline is a suite of plugins which supports implementing and integrating continuous delivery pipelines into Jenkins.
* The definition of a Jenkins Pipeline is typically written into a text file (called a Jenkinsfile) which in turn is checked into a project’s source control repository.
* Steps for creating Jenkins Pipeline Job:
   1. Click the New Item menu within Jenkins.
![New Item](/images/create_new_job.png)
   2. Provide a name for your new item (e.g. Tomcat-Installation) and select Pipeline.
   3. Under the pipeline section, select `Pipeline script from SCM`, choose the type of repository you want to use and fill in the details.
![Define Pipeline](/images/define_pipeline.png)
   4. Click the Save button

**3. Run the pipeline**

This pipeline demostrates multiple tomcat server installation(v9.0.1) on same machine. 
![Execution log 1](/images/execution_log_1.png)
![Execution log 2](/images/execution_log_2.png)


