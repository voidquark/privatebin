# PrivateBin - Ansible Role

[![License](https://img.shields.io/github/license/voidquark/privatebin)](LICENSE)

The Ansible PrivateBin Role empowers you to effortlessly deploy and manage a secure [PrivateBin](https://github.com/PrivateBin/PrivateBin) service using a rootless Podman container.

**🔑 Key Features**
- **🛡️ Root-less deployment**: PrivateBin is securely containerized and operates in a root-less mode within a user namespace. The container is managed through a systemd unit.
- **🔄 Idempotent deployment**: Role embraces idempotent deployment, ensuring that the state of your deployment always matches your desired inventory.
- **📦 Out-of-the-box Deployment**: Get Privatebin up and running quickly with default configurations that work seamlessly with Red Hat family systems. See [Quick Start](#quick-start) for easy setup.
- **🧩 Flexible Configuration**: Easily customize Privatebin configuration to match your specific requirements.

📢 **[Check the blog post](https://voidquark.com/blog/privatebin-deployment-with-rootless-podman-using-ansible-role/)** 📝 **Understand the rationale behind constructing this role in a specific manner.**

## Table of Content

- [Requirements](#requirements)
- [Role Variables](#role-variables)
- [Playbook](#playbook)
- [Quick Start](#quick-start)

## Requirements

- Ansible 2.10+
- Tested on `RHEL`/`RockyLinux` 9, but should work with compatible distributions.
- If the following Ansible collections are not already available in your environment, please install them: `ansible-galaxy collection install ansible.posix` and `ansible-galaxy collection install containers.podman`.
- Ensure that the `podman` and `loginctl` binaries are present on the target system.

## Role Variables

```yaml
privatebin_user: "privatebin"
```
OS user under which the PrivateBin container runs.

```yaml
privatebin_group: "privatebin"
```
OS group for the PrivateBin user.

```yaml
privatebin_dir: "/home/{{ privatebin_user }}/privatebin"
```
Default PrivateBin directory where all templates and configuration files are stored by Ansible.

```yaml
privatebin_data_dir: "{{ privatebin_dir }}/data"
```
Default PrivateBin data directory.

```yaml
privatebin_container_name: "privatebin"
```
Default name of the PrivateBin container.

```yaml
privatebin_container_image: "docker.io/privatebin/nginx-fpm-alpine:stable"
```
Default container image.

```yaml
privatebin_container_volumes:
  - "{{ privatebin_data_dir }}:/srv/data:rw,Z"
  - "{{ privatebin_dir }}/conf.php:/srv/cfg/conf.php:ro,Z"
```
By default, only the data directory and PrivateBin configuration file are mounted as container volumes. If you need to modify the `php.ini` or `nginx` configuration, you will need to append additional volumes.

```yaml
privatebin_container_publish: "8080:8080"
```
Default container port configuration is set to `8080` for both the host and container.

```yaml
privatebin_conf_raw: |
  ;<?php http_response_code(403); /*
  ; config file for PrivateBin
  ;
  ; An explanation of each setting can be find online at https://github.com/PrivateBin/PrivateBin/wiki/Configuration.

  [main]
  discussion = true
  opendiscussion = false
  password = true
  fileupload = false
  burnafterreadingselected = false
  defaultformatter = "plaintext"
  sizelimit = 10485760
  template = "bootstrap5"
  languageselection = false

  [expire]
  default = "1week"

  [expire_options]
  5min = 300
  10min = 600
  1hour = 3600
  1day = 86400
  1week = 604800
  1month = 2592000
  1year = 31536000
  never = 0

  [formatter_options]
  plaintext = "Plain Text"
  syntaxhighlighting = "Source Code"
  markdown = "Markdown"

  [traffic]
  limit = 10

  [purge]
  limit = 300
  batchsize = 10

  [model]
  class = Filesystem
  [model_options]
  dir = PATH "data"

  [yourls]
```
The default PrivateBin PHP configuration. You can customize this as needed. For detailed configuration options, refer to the [PrivateBin documentation](https://github.com/PrivateBin/PrivateBin/wiki/Configuration).

### Variables not set by default

```yaml
privatebin_firewalld_expose_port: 8080
```
This variable specifies the TCP port for exposing Privatebin via Firewalld. By default, it is not exposed.

```yaml
privatebin_custom_conf:
  - filename: "php.ini"
    raw_content: |
      EXAMPLE
  - filename: "site.conf"
    raw_content: |
      EXAMPLE
```
You can template multiple custom configuration files, such as `php.ini` or any `nginx` config. These files are templated in the `privatebin_dir`, and the file name is specified using the `filename` attribute. Please note you must update the `privatebin_container_volumes` variable to include your custom configs, which need to be mounted inside the container.


## Dependencies

No Dependencies

## Playbook

- Example playbook to deploy privatebin
```yaml
- name: Manage Privatebin service
  hosts: privatebin
  gather_facts: false
  become: true
  roles:
    - role: voidquark.privatebin
```

## Quick Start

To quickly deploy Privatebin using this Ansible role, follow these steps:

**1. Set up your project directory structure:**
```shell
ansible_structure
├── playbook
│   └── function_privatebin_deploy.yml # Playbook
└── inventory
    ├── group_vars
    │   └── privatebin
    │       └── privatebin_vars.yml # Overwrite variables in group_vars (optional)
    ├── hosts
    └── host_vars
        └── privatebin.voidquark.com
            └── host_vars.yml # Overwrite variables in host_vars (optional)
```
**2. Install the Ansible Privatebin Role from Ansible Galaxy:**
```shell
ansible-galaxy install voidquark.privatebin
```
**3. Create your inventory - `inventory/hosts`**
```shell
[privatebin]
privatebin.voidquark.com
```
**4. Create your playbook - `playbook/function_privatebin_deploy.yml`**
```yaml
- name: Manage Privatebin service
  hosts: privatebin
  gather_facts: false
  become: true
  roles:
    - role: voidquark.privatebin
```
**5. Execute the playbook**
```shell
# Deployment
ansible-playbook -i inventory/hosts playbook/function_privatebin_deploy.yml
```

## License

MIT

## Contribution

Feel free to customize and enhance the role according to your needs.
Your feedback and contributions are greatly appreciated. Please open an issue or submit a pull request with any improvements.
