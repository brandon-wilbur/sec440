# Availability and Redundancy Security Reflection

## Problem Area 1: Cleartext passwords to WordPress

Passwords are transmitted in the clear back and forth from WordPress when logging in to make posts or administer changes through the web GUI.

### Solution

Create a certificate authority for the network and issue certificates to web servers.

## Problem Area 2: No authentication between VRRP addresses

There is no authentication currently in place that dictates what devices can service traffic to virtual IP addresses in this case.

### Solution

Create long passphrases for each virtual IP instance. This could be implemented on VyOS's virtual interfaces and on Keepalived instances that are used for web servers and database servers.

## Problem Area 3: Firewall auditing

There was no heavy auditing of firewall rules as redundancy was created. Though they were locked down the first week, they were opened up to allow some traffic that may not be necessary. These rules should be audited and tested in the environment to ensure only necessary services are allowed.

