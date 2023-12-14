# Ansible role: labocbz.add_add_haproxy_bdd_confs_bdd_confs

![Licence Status](https://img.shields.io/badge/licence-MIT-brightgreen)
![CI Status](https://img.shields.io/badge/CI-success-brightgreen)
![Testing Method](https://img.shields.io/badge/Testing%20Method-Ansible%20Molecule-blueviolet)
![Testing Driver](https://img.shields.io/badge/Testing%20Driver-docker-blueviolet)
![Language Status](https://img.shields.io/badge/language-Ansible-red)
![Compagny](https://img.shields.io/badge/Compagny-Labo--CBZ-blue)
![Author](https://img.shields.io/badge/Author-Lord%20Robin%20Cbz-blue)

## Description

![Tag: Ansible](https://img.shields.io/badge/Tech-Ansible-orange)
![Tag: Debian](https://img.shields.io/badge/Tech-Debian-orange)
![Tag: HAProxy](https://img.shields.io/badge/Tech-HAProxy-orange)
![Tag: Databases](https://img.shields.io/badge/Tech-Databases-orange)
![Tag: Load balancer](https://img.shields.io/badge/Tech-Load%20balancer-orange)

An Ansible role create and add HAProxy BDD confs to your server.


This role is a variant of the previous one, but it focuses on creating HAProxy configurations tailored for database services.

Similar to the previous role, this variant also operates through a set of YAML-based configuration objects to define different HAProxy configurations for various database services.

For each database service, the role allows administrators to specify a unique name. The frontend configuration includes essential details such as descriptions, bind addresses, and ports. Notably, it uses the TCP mode instead of the HTTP/HTTPS mode used for web services.

The backend configuration offers options like load balancing mode (e.g., round-robin, least connections), TCP keep-alive (tcpka), and defining multiple backend servers. Each backend server represents a database server to which HAProxy will forward requests.

In the case of clustered databases, administrators can define multiple backend servers for failover scenarios. The role includes a "backup" option for certain servers, ensuring that they serve as backup nodes to provide high availability in case the primary servers become unavailable.

In summary, this role extends the capabilities of the previous HAProxy role to handle database services specifically. By leveraging YAML-based configuration objects, administrators can easily define various parameters to create customized HAProxy configurations for their database setups. This flexibility enables the role to support a wide range of database clustering and failover scenarios, providing robust and high-performing load balancing for database services.

## Folder structure

By default Ansible will look in each directory within a role for a main.yml file for relevant content (also man.yml and main):

```PYTHON
.
├── README.md  # Contains an overview of the role and its purpose.
├── defaults
│   ├── main.yml  # Contains default variables for the role that can be overridden by users.
│   └── README.md  # Contains documentation for the default variables.
├── files
│   └── README.md  # Contains documentation for the files in the directory.
├── handlers
│   ├── main.yml  # Contains handlers that can be called by tasks within the role.
│   └── README.md  # Contains documentation for the handlers.
├── meta
│   ├── main.yml  # Contains metadata about the role, including dependencies and supported platforms.
│   └── README.md  # Contains documentation for the role metadata.
├── tasks
│   ├── main.yml  # Contains tasks to be executed by the role on the managed nodes.
│   └── README.md  # Contains documentation for the tasks.
├── templates
│   └── README.md  # Contains documentation for the templates.
└── vars
    ├── main.yml  # Contains variables that are specific to the role and are not meant to be overridden.
    └── README.md  # Contains documentation for the role variables.
```

## Tests and simulations

### Basics

You have to run multiples tests. *tests with an # are mandatory*

```MARKDOWN
# lint
# syntax
# converge
# idempotence
# verify
side_effect
```

Executing theses test in this order is called a "scenario" and Molecule can handle them.

Molecule use Ansible and pre configured playbook to create containers, prepare them, converge (run the role) and verify its execution.
You can manage multiples scenario with multiples tests in order to get a 100% code coverage.

This role contains a ./tests folder. In this folder you can use the inventory or the tower folder to create a simualtion of a real inventory and a real AWX / Tower job execution.

### Command reminder

```SHELL
# Check your YAML syntax
yamllint -c ./.yamllint .

# Check your Ansible syntax and code security
ansible-lint --config=./.ansible-lint .

# Execute and test your role
molecule lint
molecule create
molecule list
molecule converge
molecule verify
molecule destroy

# Execute all previous task in one single command
molecule test
```

## Installation

To install this role, just copy/import this role or raw file into your fresh playbook repository or call it with the "include_role/import_role" module.

## Usage

### Vars

Some vars a required to run this role:

```YAML
---
add_haproxy_bdd_confs_confs_path: "/etc/haproxy/conf.d"
add_haproxy_bdd_confs_ssl_path: "/etc/haproxy/ssl"

add_haproxy_bdd_confs_haproxy_group: "haproxy"
add_haproxy_bdd_confs_haproxy_user: "haproxy"

add_haproxy_bdd_confs_configurations:
  - name: "my.database.domain.tld"
    frontend:
      description: "My first database with TCP frontend address"
      bind: "127.0.0.1:10030"
      ssl: true
      crt: "{{ add_haproxy_bdd_confs_ssl_path }}/my.https.database.domain.tld/my.https.database.domain.tld.pem.crt"
      key: "{{ add_haproxy_bdd_confs_ssl_path }}/my.https.database.domain.tld/my.https.database.domain.tld.pem.key"
      mode: "tcp"
    backend:
      balance: leastconn
      options:
        - tcpka
      servers:
        - name: "backend-server-1"
          addresse: "127.0.0.1"
          port: "3306"

  - name: "my.cluster.database.domain.tld"
    frontend:
      description: "My first cluster database with TCP frontend address"
      bind: "127.0.0.1:10040"
      mode: "tcp"
    backend:
      balance: roundrobin
      options:
        - tcpka
      servers:
        - name: "backend-server-2-1"
          addresse: "127.0.0.1"
          port: "3306"
        - name: "backend-server-2-1"
          addresse: "127.0.0.1"
          port: "3306"
        - name: "backend-server-3-1"
          addresse: "127.0.0.1"
          port: "3306"

  - name: "my.clusterFailover.database.domain.tld"
    frontend:
      description: "My first cluster database with TCP frontend address and FAILOVER"
      bind: "127.0.0.1:10040"
      mode: "tcp"
    backend:
      balance: roundrobin
      options:
        - tcpka
      servers:
        - name: "backend-server-2-1"
          addresse: "127.0.0.1"
          port: "3306"
        - name: "backend-server-2-1"
          addresse: "127.0.0.1"
          port: "3306"
          backup: true
        - name: "backend-server-3-1"
          addresse: "127.0.0.1"
          port: "3306"
          backup: true
```

The best way is to modify these vars by copy the ./default/main.yml file into the ./vars and edit with your personnals requirements.

You can set vars in the template model in Ansible AWX / Tower or just surchage them during the playbook call.

In order to surchage vars, you have multiples possibilities but for mains cases you have to put vars in your inventory and/or on your AWX / Tower interface.

```YAML
# From inventory
---
inv_add_haproxy_bdd_confs_confs_path: "/etc/haproxy/conf.d"
inv_add_haproxy_bdd_confs_ssl_path: "/etc/haproxy/ssl"

inv_add_haproxy_bdd_confs_configurations:
  - name: "my.database.domain.tld"
    frontend:
      description: "My first database with TCP frontend address"
      bind: "127.0.0.1:10030"
      ssl: true
      crt: "{{ inv_add_haproxy_bdd_confs_ssl_path }}/my.https.database.domain.tld/my.https.database.domain.tld.pem.crt"
      key: "{{ inv_add_haproxy_bdd_confs_ssl_path }}/my.https.database.domain.tld/my.https.database.domain.tld.pem.key"
      mode: "tcp"
    backend:
      balance: leastconn
      options:
        - tcpka
      servers:
        - name: "backend-server-1"
          addresse: "127.0.0.1"
          port: "3306"

  - name: "my.cluster.database.domain.tld"
    frontend:
      description: "My first cluster database with TCP frontend address"
      bind: "127.0.0.1:10040"
      mode: "tcp"
    backend:
      balance: roundrobin
      options:
        - tcpka
      servers:
        - name: "backend-server-2-1"
          addresse: "127.0.0.1"
          port: "3306"
        - name: "backend-server-2-1"
          addresse: "127.0.0.1"
          port: "3306"
        - name: "backend-server-3-1"
          addresse: "127.0.0.1"
          port: "3306"

  - name: "my.clusterFailover.database.domain.tld"
    frontend:
      description: "My first cluster database with TCP frontend address and FAILOVER"
      bind: "127.0.0.1:10040"
      mode: "tcp"
    backend:
      balance: roundrobin
      options:
        - tcpka
      servers:
        - name: "backend-server-2-1"
          addresse: "127.0.0.1"
          port: "3306"
        - name: "backend-server-2-1"
          addresse: "127.0.0.1"
          port: "3306"
          backup: true
        - name: "backend-server-3-1"
          addresse: "127.0.0.1"
          port: "3306"
          backup: true
```

```YAML
# From AWX / Tower
---

```

### Run

To run this role, you can copy the molecule/default/converge.yml playbook and add it into your playbook:

```YAML
- name: "Include labocbz.add_add_haproxy_bdd_confs_bdd_confs"
    tags:
    - "labocbz.add_add_haproxy_bdd_confs_bdd_confs"
    vars:
    add_haproxy_bdd_confs_confs_path: "{{ inv_add_haproxy_bdd_confs_confs_path }}"
    add_haproxy_bdd_confs_configurations: "{{ inv_add_haproxy_bdd_confs_configurations }}"
    ansible.builtin.include_role:
    name: "labocbz.add_add_haproxy_bdd_confs_bdd_confs"
```

## Architectural Decisions Records

Here you can put your change to keep a trace of your work and decisions.

### 2023-05-04: First Init

* First init of this role with the bootstrap_role playbook by Lord Robin Crombez

### 2023-10-06: New CICD, new Images

* New CI/CD scenario name
* Molecule now use remote Docker image by Lord Robin Crombez
* Molecule now use custom Docker image in CI/CD by env vars
* New CICD with needs and optimization

### 2023-10-10: New SSL handling

* You can now provide custom key and certs
* Use the latest version of HAproxy role

### 2023-12-14: System users

* Role can now use system users and address groups

## Authors

* Lord Robin Crombez

## Sources

* [Ansible role documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html)
* [Ansible Molecule documentation](https://molecule.readthedocs.io/)
