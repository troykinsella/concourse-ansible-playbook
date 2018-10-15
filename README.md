# concourse-ansible-playbook

A [Concourse CI](https://concourse-ci.org) resource for running Ansible playbooks.

## Source Configuration

Most source attributes map directly to `ansible-playbook` options. See the
`ansible-playbook --help` for further details.

The `git_*` attributes are relevant to referencing git repositories in the `requirements.yml` file
which are pulled from during `ansible-galaxy install`.

* `debug`: Optional. Boolean. Default `false`. Echo command output.
* `git_global_config`: Optional. A list of git global configurations to apply (with `git config --global`).
* `git_https_username`:  Optional. The username for git http/s access.
* `git_https_password`: Optional. The password for git http/s access.
* `git_private_key`: Optional. The git ssh private key.
* `git_skip_ssl_verification`: Optional. Boolean. Default `false`. Don't verify TLS certificates.
* `remote_user`: Optional. Connect to the remote system with this user.
* `requirements`: Optional. Default `requirements.yml`. If this file is present in the 
  playbook source directory, it is used with `ansible-galaxy --install` before running the playbook.
* `ssh_common_args`: Optional. Specify options to pass to `ssh`. 
* `ssh_private_key`: Required. The `ssh` private key with which to connect to the remote system.
* `vault_password`: Optional. The value of the `ansible-vault` password.
* `verbose`: Optional. Specify, `v`, `vv`, etc., to increase the verbosity of the
  `ansible-playbook` execution.

### Example

```yaml
resource_types:
- name: ansible-playbook
  type: docker-image
  source:
    repository: troykinsella/concourse-ansible-playbook-resource
    tag: latest

resources:
- name: ansible
  type: ansible-playbook
  source:
    debug: false
    remote_user: ubuntu
    ssh_private_key: ((ansible_ssh_private_key))
    vault_password: ((ansible_vault_password))
    verbose: v
```

## Behaviour

### `check`: No Op

### `in`: No Op

### `out`: Execute `ansible` Playbook

Execute `ansible-playbook` against a given playbook and inventory file,
firstly installing dependencies with `ansible-galaxy --install` if necessary.

#### Parameters

Most parameters map directly to `ansible-playbook` options. See the
`ansible-playbook --help` for further details.

* `become`: Optional. Boolean. Default `false`. Run operations as `become` (privilege escalation).
* `become_user`: Optional. Run operations with this user.
* `become_method`: Optional. Privilege escalation method to use.
* `check`: Optional. Boolean. Default `false`. Don't make any changes; 
  instead, try to predict some of the changes that may occur.
* `diff`: Optional. Boolean. Default `false`. When changing (small) files and 
  templates, show the differences in those files; works great with `check: true`.
* `env`: Optional. A list of environment variable exports to apply.
  Useful for supplying `AWS_ACCESS_KEY_ID`, etc., for example.
* `inventory`: Required. The path to the inventory file to use, relative
  to `work_dir`.
* `playbook`: Optional. Default `site.yml`. The path to the playbook file to run,
  relative to `work_dir`.
* `vars`: Optional. An object of extra variables to pass to `ansible-playbook`.
  Mutually exclusive with `vars_file`.
* `vars_file`: Optional. A file containing a JSON object of extra variables
  to pass to `ansible-playbook`. Mutually exclusive with `vars`.
* `path`: Required. The path to the directory containing playbook sources. This typically
  will point to a resource pulled from source control.

#### Example

```yaml
# Extends example in Source Configuration

jobs:
- name:
  plan:
  - get: master # git resource
  - put: ansible
    params:
      check: true
      diff: true
      inventory: inventory/some-hosts.yml
      playbook: provision-frontend.yml
      path: master
```

## Testing

```bash
docker build .
```
