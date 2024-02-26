# Bulk AAEP - EPG binding code
## _When a performance boost is desirable_

This directory contains a playbook and a Jinja2 template. The idea here is to compute one large JSON payload that then gets pushed (POST) to APIC in one go.
The net effect is a significant performance boost compared to running a native module with a loop, because a single POST is all that is required.
The playbook works like the Postman browser plugin. It uses built-in modules (URI mostly) to log into APIC and POST content.
 
What you need:

- an inventory file for your ACI fabric with aci_username and aci_password populated
- an Ansible installation that can process Jinja2 templates
- an Ansible installation that can parse JSON

## _How to use this code_

Just run the playbook after adapting it for your Ansible inventory and desired AAEP/EPG bindings.

## _Notes__

The template is fairly generic meaning you can adapt it to a variety of use cases. As long as structure your data in a way that the template can process,
you can create virtually any kind of bulk operation against ACI while realizing a significant performance improvement. The drawback is that code readability is sacrificed because all logic has now moved into a large JSON blob.

## _Disclaimer and license_

All code found in this repo is provided as-is and subject to all terms and conditions of [the unlicense](https://unlicense.org/)
