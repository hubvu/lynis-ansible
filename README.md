## Ansible Role - Lynis Security Auditing

<p align="center">
  <a href="https://github.com/hubvu/lynis-ansible/actions" alt="Ansible Lint">
    <img src="https://github.com/hubvu/lynis-ansible/actions/workflows/ansible-lint.yml/badge.svg?branch=main">
  </a>
  <a href="https://github.com/hubvu/lynis-ansible/releases" alt="GitHub Release">
    <img src="https://img.shields.io/github/v/release/hubvu/lynis-ansible.svg">
  </a>
  <a href="./LICENSE.md" alt="MIT License">
    <img src="https://img.shields.io/badge/license-MIT-green.svg">
  </a>
  <a href="https://github.com/hubvu/lynis-ansible/tree/main#supported-distributions" alt="Supported Distributions">
    <img src="https://img.shields.io/badge/platform-debian%20%7C%20centos-lightgrey.svg">
  </a>
  <a href="https://img.shields.io/github/repo-size/hubvu/lynis-ansible.svg" alt="Repository Size">
    <img src="https://img.shields.io/github/repo-size/hubvu/lynis-ansible.svg">
  </a>
  <a href="https://img.shields.io/github/directory-file-count/hubvu/lynis-ansible.svg" alt="Repository File Count">
    <img src="https://img.shields.io/github/directory-file-count/hubvu/lynis-ansible.svg">
  </a>
</p>

- [Ansible Role - Lynis Security Auditing](#ansible-role---lynis-security-auditing)
  - [What is this?](#what-is-this)
  - [Resource Requirements](#resource-requirements)
  - [Dependencies](#dependencies)
  - [Supported Distributions](#supported-distributions)
  - [Quick-start & Usage](#quick-start--usage)
  - [Contributing](#contributing)
  - [Acknowledgements](#acknowledgements)
  - [License](#license)

### What is this?

* A set of Ansible roles for CentOS and Debian hosts that provides users with the option to deploy Lynis, run a system audit and remove the audit tool.
  * **Deploy** - [`centos_lynis.yaml`](./centos_lynis.yaml) and [`debian_lynis.yaml`](./debian_lynis.yaml) playbooks will install the latest version of Lynis available.
  * **Audit** - [`lynis_run.yaml`](./lynis_run.yaml) playbook conducts a standard system audit using the default profiles that come with Lynis. Once the audit is completed, a task will fetch the results file on the hosts `/var/log/lynis.log` and provide a copy under a `lynis_audit_results` directory for review (the directory will be created if it does not exist under the playbook directory).
  * **Remove** - [`centos_lynis_remove.yaml`](./centos_lynis_remove.yaml) and [`debian_lynis_remove.yaml`](./debian_lynis_remove.yaml) will remove Lynis from the hosts where it is deployed.
* For reference, below is a demonstration of how the directory structure of `lynis_audit_results` will look like after a number of `lynis_run.yaml` playbook runs at different time intervals.

```yaml
.
├── <INVENTORY_HOSTNAME_001>
│   ├── 2021-07-25T11:59:10Z-CentOS-8.4
│   │   └── lynis.log
│   ├── 2021-07-25T12:02:15Z-CentOS-8.4
│   │   └── lynis.log
│   ├── 2021-07-25T12:04:52Z-CentOS-8.4
│   │   └── lynis.log
│   └── 2021-07-25T12:07:18Z-CentOS-8.4
│       └── lynis.log
└── <INVENTORY_HOSTNAME_002>
    ├── 2021-07-25T11:59:10Z-Debian-10
    │   └── lynis.log
    ├── 2021-07-25T12:02:15Z-Debian-10
    │   └── lynis.log
    ├── 2021-07-25T12:04:52Z-Debian-10
    │   └── lynis.log
    └── 2021-07-25T12:07:18Z-Debian-10
        └── lynis.log
```

### Resource Requirements

* [Debian](https://www.debian.org/distrib/) and/or [CentOS Stream](https://www.centos.org/centos-stream/) host(s) that the playbooks will be run against.

### Dependencies

* `lynis`, `python3-apt`, `apt-transport-https`, `ca-certificates`, `gpg`, `curl`, `nss`, `openssl`
* [`ansible-vault`](https://docs.ansible.com/ansible/latest/user_guide/vault.html) - [**optional**] - can be used in the [`debian_ssh.yaml`](./debian_ssh.yaml) or [`centos_ssh.yaml`](./centos_ssh.yaml) playbook to encrypt and store sensitive data "at rest". 
  * In this use case, the `ansible_sudo_password` variable, which is used as the privilege escalation password, is stored in a vault.
  * Once the secret has been created and added to the playbook, in order for a user be able to become `sudo` to run the playbook, they will need to decrypt the vault to access the variable.
  * This can be achieved by passing one of the following flags listed below when executing the the playbook;
    * `--ask-vault-pass` 
    * `--vault-password-file`
  * Below is a demonstration of how the encrypted variable is defined in the playbooks;

```yaml
---
# playbook for the lynis_run role.
- hosts: centos_hosts:debian_hosts
  vars_files:
    - become-secret
  become: true
  roles:
    - lynis_run
```

  * For more information on how to create encrypted variables, review the [official `ansible` documentation](https://docs.ansible.com/ansible/latest/user_guide/vault.html#encrypting-individual-variables-with-ansible-vault).

### Supported Distributions

* Tested on;
  * `debian-10` , `centos-8-stream`

### Quick-start & Usage

* **Note** - For the audit task to run a full check, `root` privilege escalation is required and is acheived through the [`become:yes`](https://docs.ansible.com/ansible/latest/user_guide/become.html) directive.
  * To review the task, check the [`/roles/lynis_run/tasks/main.yml`](./roles/lynis_run/tasks/main.yml) for more details. 

```bash
# clone the repository
$ git clone git@github.com:hubvu/lynis-ansible.git

# navigate into the directory
$ cd lynis-ansible/

# run the master playbook `site.yaml` with verbosity
# for non Ansible Vault users
$ ansible-playbook site.yaml \
  --inventory-file=hosts \
  --ask-become-pass \
  --verbose

# run the master playbook `site.yaml` with verbosity
# for Ansible Vault users
$ ansible-playbook site.yaml \
  --inventory-file=hosts \
  --ask-vault-pass \
  --verbose

# run the playbook `lynis_run.yaml` with verbosity
$ ansible-playbook lynis_run.yaml \
  --inventory-file=hosts \
  --ask-become-pass \
  --verbose

# review the `lynis_audit_results` directory for the audit results.
$ cd lynis_audit_results
$ tree
$ cat /<inventory_hostname>/<date_time>-<distribution_name>-<distribution_version>/lynis.log
```

### Contributing

* Contribution guidelines for this project can be found in the [Contributing](./CONTRIBUTING.md) document.

### Acknowledgements

* [Lynis](https://github.com/CISOfy/Lynis).
* [Ansible Lint](https://github.com/ansible-community/ansible-lint).
* [Ansible Lint for GitHub Action](https://github.com/ansible/ansible-lint-action).

### License

* Licenced under the [MIT License](./LICENSE.md).
