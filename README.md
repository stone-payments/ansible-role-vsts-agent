# Stone.Co VSTS Agent

This is an Ansible role to install Azure VSTS Agent on Linux or Windows. Read more about VSTS agents [here](https://docs.microsoft.com/en-us/vsts/build-release/concepts/agents/agents).

## What does this project do?

- Install/uninstall vsts-agent on debian/rhel family and windows family
- Configure vsts-agent (supported modes: Environment/Deployment Group/Agent Pool)

## Role Variables

Default values are placed in [`defaults/main.yml`](defaults/main.yml).

- **vsts_accountname**: Your VSTS account name. The name before `visualstudio.com` in your access url.
- **vsts_accesstoken**: Your personal access token to register the agent.
- **vsts_projectname**: The VSTS Team Project to register the agent.

## Setup

**User must choice one agent type and set its variable:**

- **vsts_environmentname**: The Environment virtual machine target to register the agent.
- **vsts_poolname**: The Agent Pool to register the agent.
- **vsts_deploymentgroupname**: The Deployment Group to register the agent.

### Observation for Linux hosts

The agent run as `vsts_agent_user`, by default defined as `vsts`. This user has limited permissions at sudoers, but users can override agent permissions throught `vsts_sudo_permissions` variable.

Ex:

```yaml
vsts_sudo_permissions:
  - /bin/systemctl reload vsts-agent
  - /bin/systemctl status vsts-agent
  - /bin/systemctl start vsts-agent
  - /bin/systemctl restart vsts-agent
  - /bin/systemctl stop vsts-agent
  - /bin/systemctl daemon-reload
```

This role creates the directories `vsts_work_folder` to receive the artifacts and `vsts_apps_dir`, expected to be the final destiny for the applications files. The agent has full permission inside these directories. You should create the applications directories and deploy files inside `vsts_apps_dir`, avoiding sudo permissions as possible.
By default: `vsts_apps_dir: ""`.

## Installation

We usually install roles with ansible galaxy.

Adds the role to your `requirements.yml` file

```yaml
- src: git+git@github.com:stone-payments/ansible-role-vsts-agent.git
  version: master
  name: stone-payments.vsts-agent
```

Then run:

```bash
ansible-galaxy install -r requirements.yml
```

## Playbook Example

### Example Playbook to install on Linux

```yaml
- hosts: linux
  become: true
  roles:
    - role: stone-payments.vsts-agent
      vars:
        vsts_accountname: accountname
        vsts_environmentname: "AppName-Production-DC1"
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
      vars_prompt:
      - name: "vsts_accesstoken"
        prompt: "Enter vsts_accesstoken"
        private: yes
```

### Example Playbook to install deployment agent in deployment group

```yaml
- hosts: windows
  vars:
    vsts_accountname: myorg # For https://dev.azure.com/myorg
    vsts_projectname: MyTeam
    vsts_deploymentgroupname: MyApp-Staging
  vars_prompt:
  - name: "vsts_accesstoken"
    prompt: "Enter vsts_accesstoken"
    private: yes
  roles:
    - { role: stone-payments.vsts-agent }
```

### Example Playbook to install deployment agent on environment

```yaml
- hosts: windows
  vars:
    vsts_accountname: myorg # For https://dev.azure.com/myorg
    vsts_projectname: MyTeam
    vsts_environmentname: MyApp-Staging
  vars_prompt:
  - name: "vsts_accesstoken"
    prompt: "Enter vsts_accesstoken"
    private: yes
  roles:
    - { role: stone-payments.vsts-agent }
```

### Example Playbook to install queue agent

```yaml
- hosts: all
  vars:
    vsts_accountname: myorg # For https://dev.azure.com/myorg
    vsts_projectname: MyTeam
    vsts_poolname: WinBuilders
  vars_prompt:
  - name: "vsts_accesstoken"
    prompt: "Enter vsts_accesstoken"
    private: yes
  roles:
    - { role: stone-payments.vsts-agent }
```

### Example Playbook to uninstall agent

```yaml
- hosts: all
  vars:
    vsts_accountname: myorg # For https://dev.azure.com/myorg
    vsts_remove: true
  vars_prompt:
  - name: "vsts_accesstoken"
    prompt: "Enter vsts_accesstoken"
    private: yes
  roles:
    - { role: stone-payments.vsts-agent }
```

### Example Playbook to reinstall deployment agent

```yaml
- hosts: all
  vars:
    vsts_accountname: myorg # For https://dev.azure.com/myorg
    vsts_projectname: MyTeam
    vsts_deploymentgroupname: MyApp-Staging
    vsts_reinstall: true
  vars_prompt:
  - name: "vsts_accesstoken"
    prompt: "Enter vsts_accesstoken"
    private: yes
  roles:
    - { role: stone-payments.vsts-agent }
```

### Advanced scenarios

#### How to install multiple queue agents on the same machine (not supported for deployment group agents)

In your inventory add multiple entries for the same host with different values for vsts_agent_name and vsts_agentfolder

```ini
[queue-agents]
HOST-01-1 ansible_host=10.0.0.10 vsts_agent_name=HOST-01-1 vsts_agentfolder=C:\\vstsagent\\A1
HOST-01-2 ansible_host=10.0.0.10 vsts_agent_name=HOST-01-2 vsts_agentfolder=C:\\vstsagent\\A2
...
```

## Testing

This role implements unit tests with [`molecule`](https://molecule.readthedocs.io/en/latest/) and [`testinfra`](https://testinfra.readthedocs.io/en/latest/). Notice that we only support Molecule 3.0 or greater. You can install molecule with:

```shell
pipenv install --dev --three
```

After having Molecule setup, you can run the tests with this steps:

```sh
molecule test [-s scenario_name]
```

## Contributing

See the [CONTRIBUTING.md](./CONTRIBUTING.md) and just open a PR. We love PRs!
