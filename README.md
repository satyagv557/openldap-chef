
1.       Using AWS or another cloud provider, create and deploy a running instance of an LDAP server using a configuration management tool.

This scenario Im achieving with Chef managed hosting with an Instance in AWS. Please find the cookbook.


2.       In the created instance, add a group named “techops_dba” to /etc/security/access.conf and /etc/sudoers. This may be done before or after the instance starts.

Usage if access.conf:
It will help us to allow or restrict user database to login and we can do restriction on IP base of login too 

We can archive this via userdata option while launching an Instance in AWS. A script which does this is below.

#!/bin/bash
groupadd techops_dba
echo "-:techops_dba:ALL" >> /etc/security/access.conf
echo "techops_dba ALL=(ALL) ALL" >>  /etc/sudoers
end

3 .   What is the NTP stratum of the created host? What is an acceptable load average threshold for the host?

NTP stratum: 
stratum levels define the distance from the reference clock.  A reference clock is a stratum-0 device that is assumed to be accurate and has lttle or no delay associated with it.

we can get details related to it from Instance by below commands.

 #ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*luna.jake-tm.co 156.107.125.80   3 u  949 1024  377   10.684   -0.786   0.684
 LOCAL(0)        .LOCL.           5 l  20d   64    0    0.000    0.000   0.000

#ntpq -c rv
associd=0 status=0615 leap_none, sync_ntp, 1 event, clock_sync,
version="ntpd 4.2.6p5@1.2349-o Mon Jan 25 14:08:27 UTC 2016 (1)",
processor="x86_64", system="Linux/3.2.30-49.59.amzn1.x86_64", leap=00,
stratum=4, precision=-20, rootdelay=20.129, rootdisp=101.206,
refid=151.236.19.231,
reftime=da8a19f4.e5a7e52e  Wed, Mar  9 2016  3:41:40.897,
clock=da8a1de1.04a61d46  Wed, Mar  9 2016  3:58:25.018, peer=53889,
tc=10, mintc=3, offset=-0.786, frequency=-26.306, sys_jitter=0.000,
clk_jitter=0.472, clk_wander=0.036



For Load Threshold :
Each core can handle a load average of 1.  That means that your core is at capacity (meaning there is one process running on the core, and nothing queued up waiting for it.) Think of 1 as being 100% capacity, so if you have 2 physical CPU’s and each of them have 4 cores then in grand total you have 8 cores.  Duh, right?  Here’s the point, if you have 8 cores then you can have an average CPU load of 8 and that would put you at capacity.  In this scenario, anything less than 8 would put you under capacity and anything over 8 would put you in the queue.

4.       Instantiate a second host and implement security so that clients are only able to access the first host from the second host. Use SSH for connectivity. This is commonly referred to as a “jump box” configuration.

we need to white-list 22 port from restricted instance in our case openldap instance to jumpbox server which we open 22 port to public ip's required. And we can enable some fail2ban for IPS. 

5.       Develop and apply automated tests to verify correctness of the LDAP server of step 1, the configuration of step 2, the results of the NTP findings of step 3, and the security restrictions of step 4.



openldap Cookbook
=================
Configures a server to be an OpenLDAP master, OpenLDAP replication slave, or OpenLDAP client.


Requirements
------------
### Platform
- Ubuntu 10.04+
- Debian
- FreeBSD 10
- RHEL and derivitives

### Chef
Chef version 0.10.10+ (with Ohai 0.6.12+) is required.

### Cookbooks
- openssh
- nscd
- openssl (for slave recipe)
- freebsd


Attributes
----------
Be aware of the attributes used by this cookbook and adjust the defaults for your environment where required, in `attributes/default.rb`.

### Overall install attributes
- `openldap['package_install_action']` - The action to be taken for all packages in the recipes. Defaults to :install, but can also be set to :upgrade to upgrade all packages referenced in the recipes.

### Client node attributes

- `openldap['server_uri']` - the URI of the LDAP server
- `openldap['basedn']` - basedn
- `openldap['cn']` - admin
- `openldap['server']` - the LDAP server fully qualified domain name, default `'ldap'.node['domain']`.
- `openldap['tls_enabled']` - specifies whether TLS will be used at all. Setting this to fals will result in your credentials being sent in clear-text.
- `openldap['tls_checkpeer']` - specifies whether the client should verify the server's TLS certificate. Highly recommended to set tls_checkpeer to true for production uses in order to avoid man-in-the-middle attacks. Defaults to false for testing and backwards compatibility.
- `openldap['pam_password']` - specifies the password change protocol to use. Defaults to md5.

### Server node attributes

- `openldap['schemas']` - Array of ldap schema file names to load
- `openldap['modules']` - Array of slapd modules names to load
- `openldap['slapd_type']` - master | slave
- `openldap['slapd_rid']` - unique integer ID, required if type is slave.
- `openldap['slapd_master']` - hostname of slapd master, attempts to search for slapd_type master.
- `openldap['database']` - Preferred database backend, defaults to HDB or MDB (for FreeBSD).
- `openldap['manage_ssl']` - Whether or not this cookbook manages your SSL certificates.
   If set to `true`, this cookbook will expect your SSL certificates to be in files/default/ssl and will configure slapd appropriately.
   If set to `false`, you will need to provide your SSL certificates **prior** to this recipe being run. Be sure to set `openldap['ssl_cert']` and `openldap['ssl_key']` appropriately.
- `openldap['ssl_cert']` - The full path to your SSL certificate.
- `openldap['ssl_key']` - The full path to your SSL key.
- `openldap['ssl_cert_source_cookbook']` - The cookbook to find the ssl cert.  Defaults to this cookbook
- `openldap['ssl_cert_source_path']` - The path in the cookbook to find the ssl cert file.
- `openldap['ssl_key_source_cookbook']` - The cookbook to find the ssl key.  Defaults to this cookbook
- `openldap['ssl_key_source_path']` - The path in the cookbook to find the ssl key file.
- `openldap['cafile']` - Your certificate authority's certificate (or intermediate authorities), if needed.


