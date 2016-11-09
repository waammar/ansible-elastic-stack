Ansible-Elastic-Stack
===========
Ansible Playbook for setting up the Elastic Stack and Filebeat client on remote hosts

![ELK](/image/ansible-elk.png?raw=true)

**What does it do?**
   - Automated deployment of a full Elastic Stack (Elasticsearch, Logstash, Kibana) version 5.x
     * Uses Nginx as a reverse proxy for Kibana
     * Generates SSL certificates for Filebeat or Logstash-forwarder
     * Deploys ELK clients using SSL and Filebeat for Logstash
     * All service ports can be modified in ```install/group_vars/all.yml```

**Requirements**
   - RHEL7 or CentOS7+ server/client with no modifications
     - Fedora 23 or higher needs to have ```yum python2 python2-dnf libselinux-python``` packages.
       * You can run this against Fedora clients prior to running Ansible ELK:
       - ```ansible fedora-client-01 -u root -m shell -i hosts -a "dnf install yum python2 libsemanage-python python2-dnf -y"```
   - Deployment tested on Ansible 1.9.4 and 2.0.2

**Notes**
   - Sets the nginx htpasswd to admin/admin initially
   - nginx ports default to 80/8080 for Kibana and SSL cert retrieval (configurable)
   - Uses OpenJDK for Java
   - It's fairly quick, takes around 3minutes on test VM
   - Filebeat templating is focused around OpenStack service logs

**ELK Server Instructions**
   - Clone repo and setup your hosts file
```
git clone https://github.com/waammar/ansible-elastic-stack
cd ansible-elastic-stack
sed -i 's/host-01/elkserver/' hosts
sed -i 's/host-02/elkclient/' hosts
```
   - Run the playbook
```
ansible-playbook -i hosts elk.yml
```
   - Navigate to the server at http://yourhost
   - Default login is admin/admin
![ELK](/image/elk-index.png?raw=true "Click the green button.")

**ELK Client Instructions**
   - Run the client playbook against the generated ``elk_server`` variable
```
ansible-playbook -i hosts elk-client.yml --extra-vars 'elk_server=X.X.X.X'
```

**File Hierarchy**
```
.
├── hosts
├── elk-client.yml
├── elk.yml
├── group_vars
│   └── all.yml
└── roles
    ├── elasticsearch
    │   ├── files
    │   │   ├── elasticsearch.in.sh
    │   │   └── elasticsearch.repo
    │   └── tasks
    │       └── main.yml
    ├── filebeat
    │   ├── files
    │   │   └── filebeat.repo
    │   ├── tasks
    │   │   └── main.yml
    │   └── templates
    │       └── filebeat.yml.j2
    ├── kibana
    │   ├── files
    │   │   ├── filebeat-dashboards.zip
    │   │   └── kibana.repo
    │   └── tasks
    │       └── main.yml
    ├── logstash
    │   ├── files
    │   │   ├── 01-lumberjack-input.conf
    │   │   ├── 10-syslog.conf
    │   │   ├── 10-syslog-filter.conf
    │   │   ├── 30-elasticsearch-output.conf
    │   │   ├── 30-lumberjack-output.conf
    │   │   ├── filebeat-index-template.json
    │   │   └── logstash.repo
    │   ├── tasks
    │   │   └── main.yml
    │   └── templates
    │       ├── 02-beats-input.conf.j2
    │       └── openssl_extras.cnf.j2
    └── nginx
        ├── tasks
        │   └── main.yml
        └── templates
            ├── kibana.conf.j2
            └── nginx.conf.j2

27 directories, 34 files
```
