## Usage
```
ansible-playbook InitialSetup.yaml -i aciInventory.yaml
```

## Simplified Configuration

This playbook takes a fairly simple inventory-driven approach to automating essential fabric configuration elements.
You pretty much never need to worry about the playbook itself; instead you should visit the aci_vars.yaml file. 
That file describes a series of desired ACI configuration items in a hierarchical format.
Consider the following example:

```
aci_data:
  system: # System Settings
    apic_conn_pref: "ooband"

    system_alias_banner:
      alias: "Ansible-demo"
      apic_banner: "Welcome to this Ansible-automated fabric"
      switch_banner: "You are logging into a switch that belongs to an Ansible-automated ACI fabric"
```

The playbook first loads the entire aci_vars.yaml file and then decides whether to run a given task or not based on the presence of a given variable.
For example, one task runs if aci_data.system.system_alias_banner.alias is defined and not null.
This means that if you omit a given variable or skip an entire block, the playbook will *not* error out on you. It will simply skip over items that aren't defined in the aci_vars.yaml file.
This is essentially a data model or inventory-driven approach to configuring ACI.

## More Explanations

### Ansible module `aci_rest`

The entire playbook uses `cisco.aci.aci_rest` to send REST API payload via Ansible directly to APIC. Some operations make use of a custom Python filter included in the filter_plugins directory.
The code is reasonably simple to understand, follow and expand should you see a need to do so.

