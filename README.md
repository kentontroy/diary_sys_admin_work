### Some notes and thoughts on ongoing System Administration tasks and DevOps / MLOps challenges

#### FreeIPA IPv6 ruminations
```
So, FreeIPA requires IPv6. As an alternative to AD, Cloudera can use FreeIPA for LDAP, as a KDC, and for DNS.
However, Cloudera doesn't support IPv6.

When using a dedicated node for FreeIPA that's only configured for IPv4, you will encounter this error:
"IPv6 stack is enabled in the kernel but there is no interface that
has ::1 address assigned. Add ::1 address resolution to 'lo' interface.
You might need to enable IPv6 on the interface 'lo' in sysctl.conf."

To avoid this, I ran (add to /etc/sysctl.conf for permanence):
sysctl net.ipv6.conf.all.disable_ipv6=0

On the Cloudera nodes, on RHEL 9, disable IPv6 from loading in the kernel on boot:
grubby --args ipv6.disable=1 --update-kernel DEFAULT
grubby --info DEFAULT
reboot
```

#### FreeIPA examples for DNS wildcards
```
Newer versions of Cloudera can deploy virtual clusters for ML and Data Engineering on Kubernetes and OpenShift.
If a customer's security standards allow it, the use of wildcard naming in DNS eases name resolution of dynamic
virtual hostnames that arise. For the examples below, assume a domain name of cloudera-lab.com.

You will encounter a dynamically created name such as console-cdp.apps.cloudera-lab.com representing the virtual
host for the Control Plane. As another example, you will see ml-c72eb255-d47.apps.cloudera-lab.com as a virtual
host for a Cloudera Machine Learning environment.

Rather than create static A records for these in DNS, consider using wildcards such as:

ipa dnszone-add --name-from-ip=192.168.1.0/24
ipa dnsrecord-add 1.168.192.in-addr.arpa. 5 --ptr-rec master-ecs-server.cloudera-lab.com.
ipa dnsrecord-add cloudera-lab.com. --cname-hostname=master-ecs-server apps
ipa dnsrecord-add cloudera-lab.com. *.apps --a-rec 192.168.1.5

The syntax above first creates a Reverse Address Zone for the subnet. The master host is then added to the zone.
FreeIPA will automatically create a Forward Zone on your behalf. Next, you create a CNAME alias that maps
apps.cloudera-lab.com to master-ecs-server.cloudera-lab.com. Finally, you create an A record that maps
*.apps.cloudera-lab.com to master-ecs-server.cloudera-lab.com.

Essentially, as Cloudera spins up multiple virtual hosts dynamically, all of them will have some name,
e.g. xl1rsx4532.apps.cloudera-lab.com, that map to a master node and can potentially float to another node in the
event of a failover.
```
#### Postgresql connectivity issue and subsequent gymanistics thereafter when upgrading Cloudera Manager
```
Cloudera Manager (CM) uses a database (in test cases, an embedded Postgresql instance) to store configuration details and
metadata for services such as SCM, Ranger, Hive, Hue, etc. When updating Cloudera Manager with the latest cumulative
hot fix, the cloudera-scm-server-db service would not start.

I forgot to turn off services that use the Postgresql instance such as Hive while performing the upgrade of
cloudera-scm-server. Removing lock files for Postgresql didn't work. So, I rebooted the Cloudera Manager node. The
cloudera-scm-server-db then started successfully. However, upon reboot, /etc/resolv.conf, 'mysteriously' changed the
nameserver reference to point to an invalid IP. This wasn't detected until receiving Java Socket connection failures
when CM tries to perform heartbeat, service monitoring, and other checks. It wasn't intuitive at first that the
connection failures were due to an inability of the CM server to resolve the hostnames of the cluster nodes to be
monitored.
```
