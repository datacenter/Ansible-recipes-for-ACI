## What this does

Fairly simple playbook that uploads switch and controller images to APIC.
It then creates two maintenance policies (one for odd switches, one for even switches) and creates one upgrade group per switch.
Either the odd or the even policy is attached to a switch upgrade group.
Users can then log on the UI and trigger the upgrade process - this playbook does not automate this.
Should you wish to fully automate the update, a few lines using the ```aci_rest``` module could do the job.
