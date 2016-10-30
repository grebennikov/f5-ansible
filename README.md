# Ansible F5

## Overview

###IMPORTANT
If using F5 playbooks on the same host where OSA is bootstrapped, make sure it is Ansible 2.2. If not:


# cd /opt
# git clone https://github.com/ansible/ansible
# cd ansible
# git checkout stable-2.2
# git submodule update --init --recursive
# pip install /opt/ansible
# pip install suds bigsuds f5-sdk


######Just in case it is necessary to switch back to Ansible 1.9:


# cd /opt/ansible_819c51cd807061845ad5db9c5aaa8e05a623939b
# git submodule update --init --recursive
# pip install /opt/ansible_819c51cd807061845ad5db9c5aaa8e05a623939b
######


After that the environment is ready to use Ansible 2.2 for F5-bigip playbooks




# git clone https://github.com/F5Networks/f5-ansible
# cd f5-ansible
# ### in inventory/hosts specify the IP of the actual F5
# ### in inventory/group_vars/all specify proper username and password for F5
# ### verify the F5 is responding to the client properly executing
# ansible-playbook -i inventory/hosts playbooks/bigip_facts.yaml


Copy dynamic_inventory.py script from OSA to ./inventory
Add the pattern to /etc/openstach_deploy/openstack_user_config:


f5_hosts:
  f5:
    ip: 10.36.190.199
Also it is necessary to add “10.36.190.199 f5” into /etc/hosts, or use proper DNS name for the balancer.


Add/replace vars in ./inventory/group_vars/all:


---


bigip_username: "adminm"
bigip_password: "password"
bigip_port: "443"
bigip_partition: "Common"
validate_certs: "no"


pool_lb_method: "round_robin"
pool_reselect_tries: 10
pool_lb_method_alt:
    - "ratio_member"
    - "least_connection_member"
    - "observed_member"
    - "predictive_member"
    - "ratio_node_address"
    - "least_connection_node_address"
    - "fastest_node_address"
    - "observed_node_address"
    - "predictive_node_address"
    - "dynamic_ratio"
    - "fastest_app_response"
    - "least_sessions"
    - "dynamic_ratio_member"
    - "l3_addr"
    - "weighted_least_connection_member"
    - "weighted_least_connection_node_address"
    - "ratio_session"
    - "ratio_least_connection_member"
    - "ratio_least_connection_node_address"
connection_limit: 100
rate_limit: 50
ratio: 2


Create necessary vars file so that it can be used together with the playbook:


---
pool_name: glance_api
#pool_members:
#   - ip: 10.0.0.1
#     desc: "node1"
#   - ip: 10.0.0.1
#     desc: "node1"
pool_member_port: 9292
vs_profiles1:
#    - tcp
    - http
node_host: "10.0.0.100" #VIP address 
vs_description: "glance-api"
vs_name: "glance-api"
vs_port1: 9292
vs_snat1: "Automap"
va_address: 10.0.0.100 #VIP address




ansible-playbook  -i inventory/dynamic_inventory.py -e @add_vars /root/f5-ansible/playbooks/bigip_testpools_dyn.yaml

#################################################################################

This repository provides the foundation for working with F5 devices and Ansible.
The architecture of the modules makes inherent use of the BIG-IP SOAP and REST
APIs as well as the tmsh API where required.

These modules are freely provided to the open source community for automating
BIG-IP device configurations using Ansible. Support for the modules is provided
on a best effort basis by the F5 community. Please file any bugs, questions or
enhancement requests using [Github Issues](https://github.com/F5Networks/f5-ansible/issues)

### Requirements

* [Ansible 2.2.0 or greater][installing]
* Advanced shell for user account enabled
* [bigsuds Python Client 1.0.4 or later][bigsuds]
* [f5-sdk Python Client, latest available][f5-sdk]

### Documentation

All documentation is hosted on [ReadTheDocs][readthedocs].

When [writing new modules][writingnew], please refer to the
[Guidelines][guidelines] document.

### Purpose

The purpose of this repository is to serve as a **staging ground** for Ansible
modules that we would prefer to have upstreamed over the course of time.

The modules in this repository **may be broken** due to experimentation
or refactoring

### Your ideas

We are curious to know what sort of modules you want to see created. If you have
a use case and can sufficiently describe the behavior you want to see, open
an issue here and we will hammer out the details.

### Support

The code provided in this repository should be considered F5 contributed, and
not F5 supported. If you are familiar with similar verbiage on DevCentral, then
you are familiar with what it means here.


[bigsuds]: https://pypi.python.org/pypi/bigsuds/
[f5-sdk]: https://pypi.python.org/pypi/f5-sdk/
[readthedocs]: https://f5-ansible.readthedocs.io/en/latest/
[guidelines]: https://f5-ansible.readthedocs.io/en/latest/development/guidelines.html
[writingnew]: https://f5-ansible.readthedocs.io/en/latest/development/writing-a-module.html
[installing]: https://f5-ansible.readthedocs.io/en/latest/usage/getting_started.html#installing-ansible
