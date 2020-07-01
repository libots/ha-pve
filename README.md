# ha-pve
ProxmoxVE custom component (adds storage sensors) based on the [native integration](https://www.home-assistant.io/integrations/proxmoxve/). Mostly a temporary solution since YAML changes are no longer allowed until I can make my [pull request](https://github.com/home-assistant/core/pull/36983) acceptable, but maybe it will stick around longer if people like it.

## Configuration

<div class='note'>
You should have at least one VM, container, or storage entry configured, else this integration won't do anything.
</div>

To use the `pve` component, add the following configuration to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
pve:
  - host: IP_ADDRESS
    username: USERNAME
    password: PASSWORD
    nodes:
      - node: NODE_NAME
        vms:
          - VM_ID
        containers:
          - CONTAINER_ID
        storage:
          - STORAGE_NAME
```

{% configuration %}
host:
  description: IP address of the Proxmox VE instance.
  required: true
  type: string
port:
  description: The port number on which Proxmox VE is running.
  required: false
  default: 8006
  type: integer
verify_ssl:
  description: Whether to do strict validation on SSL certificates. If you use a self signed SSL certificate you need to set this to false.
  required: false
  default: true
  type: boolean
username:
  description: The username used to authenticate.
  required: true
  type: string
password:
  description: The password used to authenticate. Can include the realm by appending "@<realm>"
  required: true
  type: string
realm:
  description: The authentication realm of the user.
  required: false
  default: pam
  type: string
nodes:
  description: List of the Proxmox VE nodes to monitor.
  required: true
  type: map
  keys:
    node:
      description: Name of the node
      required: true
      type: string
    vms:
      description: List of the QEMU VMs to monitor.
      required: false
      type: list
    containers:
      description: List of the LXC containers to monitor.
      required: false
      type: list
    storage:
      description: List of storage nodes to monitor.
      required: false
      type: list
{% endconfiguration %}

Example with multiple VMs and no containers:

```yaml
pve:
  - host: IP_ADDRESS
    username: USERNAME
    password: PASSWORD
    nodes:
      - node: NODE_NAME
        vms:
          - VM_ID_1
          - VM_ID_2
        storage:
          - local-lvm
```

## Binary Sensor

The integration will automatically create a binary sensor for each tracked virtual machine or container. The binary sensor will either be on if the VM's state is running or off if the VM's state is different.

The created sensor will be called `binary_sensor.NODE_NAME_VMNAME_running`.

## Proxmox Permissions

To be able to retrieve the status of VMs and containers, the user used to connect must minimally have the `VM.Audit` privilege; to retrieve the status of storage entires, the user needs the `Datastore.Audit` privilege. Below is a guide to how to configure a new user with the minimum required permissions.

### Create Home Assistant Role

Before creating the user, we need to create a permissions role for the user.

1. Click `Datacenter`
2. Open `Permissions` and click `Roles`
3. Click the `Create` button above all the existing roles
4. name the new role (e.g.,  "home-assistant")
5. Click the arrow next to privileges and select `VM.Audit` (to monitor VMs and containers) and/or `Datastore.Audit` (to monitor storage) in the dropdown
6. Click `Create`

### Create Home Assistant User

Creating a dedicated user for Home Assistant, limited to only the role just created is the most secure method. These instructions use the `pve` realm for the user. This allows a connection, but ensures that the user is not authenticated for SSH connections. If you use the `pve` realm, just be sure to add `realm: pve` to your configuration.

1. Click `Datacenter`
2. Open `Permissions` and click `Users`
3. Click `Add`
4. Enter a username (e.g., "hass")
5. Set the realm to "Proxmox VE authentication server"
 Enter a secure password (it can be complex as you will only need to copy/paste it into your Home Assistant configuration)
6. Ensure `Enabled` is checked and `Expire` is set to "never"
7. Click `Add`

### Add User Permissions to Assets

To apply the user and role just created, we need to give it permissions

1. Click `Datacenter`
2. Click `Permissions`
3. Open `Add` and click `User Permission`
4. Select "/" for the path
5. Select your Home Assistant user (`hass`)
6. Select the Home Assistant role (`home-assistant`)
7. Make sure `Propagate` is checked

