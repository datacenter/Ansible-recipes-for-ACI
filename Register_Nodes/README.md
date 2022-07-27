## Usage
```
ansible-playbook RegisterNodes.yaml -i aciInventory.yaml
```

## What this does

This playbook registers nodes (leaves and spines) to your ACI fabric. To do so, simply populate the CSV file provided in this repo with your nodes. 
Optionally should you wish to assign OOB IPv4 addresses to the nodes, just populate the ipv4_addr and ipv4_gw fields in the CSV file.
If you leave those fields blank, the playbook register the node and simply skips the IPv4 assignment step. 


