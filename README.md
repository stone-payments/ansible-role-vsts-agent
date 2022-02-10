# Stone.Co VSTS Agent

This is an Ansible role to install Azure VSTS Agent on Linux.

## What does this project do?
* install/uninstall vsts-agent on debian/rhel family
* configure vsts-agent (supported modes: Environment/Deployment Group/Agent Pool)
 
## Setup
### Environment Variables
Default values are placed in [`defaults/main.yml`](defaults/main.yml).

**user must choice one agent type and set its variable:**
- vsts_environmentname
- vsts_poolname
- vsts_deploymentgroupname

The agent run as `vsts_agent_user`, by default defined as vstsuser. This user has limited permissions at sudoers, but users can override agent permissions throught `vsts_sudo_permissions` variable.
Ex:
```
vsts_sudo_permissions:
  - /bin/systemctl reload vsts-agent
  - /bin/systemctl status vsts-agent
  - /bin/systemctl start vsts-agent
  - /bin/systemctl restart vsts-agent
  - /bin/systemctl stop vsts-agent
  - /bin/systemctl daemon-reload
```

This role creates the directories `vsts_workfolder` to receive the artifacts and `vsts_apps_dir`, expected to be the final destiny for the applications files. The agent has full permission inside these directories. You should create the applications directories and deploy files inside `vsts_apps_dir`, avoiding sudo permissions as possible.
By default: `vsts_apps_dir: /opt/apps`.

## Installation
We usually install roles with ansible galaxy.

Adds the role to your `requirements.yml` file
```
- src: git+git@github.com:stone-payments/ansible-role-vsts-agent.git
  version: master
  name: stone-payments.vsts-agent
```
Then run:
```
ansible-galaxy install -r requirements.yml
```

## Playbook Example
```
- hosts: all
  become: true
  roles:
    - role: stone-payments.vsts-agent
      vars:
        vsts_accountname: accountname
        vsts_environmentname: "AppName-Production-DC1"
        vsts_accesstoken: "yourtoken"
        vsts_projectname: "Project-Name"
        vsts_sudo_permissions:
          - /bin/systemctl reload vsts-agent
          - /bin/systemctl status vsts-agent
          - /bin/systemctl start vsts-agent
          - /bin/systemctl restart vsts-agent
          - /bin/systemctl stop vsts-agent
          - /bin/systemctl stop project-name
          - /bin/systemctl start project-name
          - /bin/systemctl restart project-name
          - /bin/systemctl daemon-reload
          - /bin/cp files/project-name.service /etc/systemd/system/project-name.service
          - /bin/systemctl enable project-name.service
          - /bin/cp files/newrelic.ini /etc/newrelic.ini
```

## Contributing
Just open a PR. We love PRs!