### A primitive but easy way to keep notes and thoughts on ongoing System Administration tasks and DevOps / MLOps challenges

#### FreeIPA IPv6 ruminations
```
So, FreeIPA requires IPv6. As an alternative to AD, Cloudera can use FreeIPA for LDAP, as a KDC, and for DNS. However, Cloudera doesn't support IPv6.

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
