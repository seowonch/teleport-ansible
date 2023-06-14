

# Remove Teleport node Service (For legacy teleport node)

### Uninstall Command ###

```
ansible-playbook -i inventories/{phase} remove_teleport.yml --extra-vars "app_name={app_name} default_user={user}"
ex. ansible-playbook -i inventories/dev remove_teleport.yml --extra-vars "app_name=teleport-ubuntu default_user=ubuntu"
```

# Install Teleport Node Service (joining-nodes-aws-iam)

An ansible role to install or update the teleport node service and teleport config on Debian and Ubuntu based systems.

Works with any architecture that teleport has a binary for, see available [teleport downloads](https://goteleport.com/teleport/download/).

Please Check the teleport config file [documentation](https://goteleport.com/docs/management/guides/joining-nodes-aws-iam/) for more information and confirm it is setup correctly.

## Requirements

Make sure legacy teleport service deleted before install the new teleport(joining-nodes-aws-iam) node.

A running teleport cluster so that you can provide the following information:

- iam token.
  -  `tctl create -f token.yaml`
  
  ```
    # token.yaml
    kind: token
    version: v2
    metadata:
    # the token name is not a secret because instances must prove that they are
    # running in your AWS account to use this token
    name: iam-token
    # set a long expiry time, the default for tokens is only 30 minutes
    expires: "3000-01-01T00:00:00Z"
    spec:
    # use the minimal set of roles required
    roles: [Node]

    # set the join method allowed for this token
    join_method: iam

    allow:
    # specify the AWS account which Nodes may join from
    - aws_account: "017915111111"
      
    ```
- address of the proxy server

## Role Variables

These are the default variables with their default values as defined in `defaults/main.yml`

```
teleport_version
```
The version of teleport to install. See [teleport downloads](https://goteleport.com/teleport/download/) for available versions.

```
teleport_architecture
```
Change `teleport_architecture` any of the following:

- `arm-bin` if you are running on ARMv7 (32-bit) based devices.
- `arm64-bin` if you are running on ARM64/ARMv8 (64-bit) based devices.
- `amd64-bin` if you are running on x86_64/AMD64 based devices.
- `386-bin` if you are running on i386/Intel based devices.

```
teleport_config_template
```
The template to use for the teleport configuration file. The default is `templates/default_teleport.yaml.j2`. It contains a basic configuration that will enable the SSH service and add a command label showing node uptime.

There are many [options available](https://goteleport.com/docs/setup/reference/config/) and you can substitute in your own template and add any variables you want.

```
teleport_service_template
```
The template to use for the teleport service file. The default is `templates/default_teleport.service.j2`. You can substitute in your own template and add any variables you want.

```
teleport_config_path
```
The path to the teleport configuration file. The default is `/etc/teleport.yaml`.

```
teleport_iam_token
```
The name for the iam token to join the AWS node. Examples are shown as defaults above (iam-token).

```
backup_teleport_config
```
Runs a backup of the teleport configuration file before overwriting it. The default is `yes`.

```
teleport_control_systemd
```
Default `yes`. Controls if this role modifies the teleport service.

```
teleport_template_config
```
Default `yes`. Controls if this role modifies the teleport config file.


## Dependencies

None

## Example Playbook
For example to install teleport on a node:
```
- hosts: all
  roles:
    - teleport
  vars:
    # optional ssh labels
    teleport_ssh_labels:
      - k: "label_key"
        v: "label_value"
    teleport_auth_server: "auth server"
    teleport_proxy_server: "proxy server"
```

*Created Teleport Config to `/etc/teleport.yaml`*

```
---
version: v3
teleport:
  auth_token: "super secret auth token"
  ca_pin: "not as secret ca pin"
  auth_server: auth server
  proxy_server: proxy server
  log:
    output: stderr
    severity: INFO
    format:
      output: text
  diag_addr: ""
ssh_service:
  enabled: "yes"
  labels:
    label_key: label_value
  commands:
  - name: hostname
    command: [hostname]
    period: 60m0s
  - name: uptime
    command: [uptime, -p]
    period: 5m0s
  - name: version
    command: [teleport, version]
    period: 60m0s
proxy_service:
  enabled: "no"
  https_keypairs: []
  https_keypairs_reload_interval: 0s
  acme: {}
auth_service:
  enabled: "no"
```

### Install Command ###

```
ansible-playbook -i inventories/{phase} install_teleport.yml --extra-vars "app_name={app_name} default_user={user}"
ex. ansible-playbook -i inventories/dev install_teleport.yml --extra-vars "app_name=teleport-ubuntu default_user=ubuntu"
```
