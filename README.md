smtp-firewall
=============

Blokuje odchozí spojení, která nepocházejí z postfixu

```
Chain OUTPUT (policy ACCEPT 25639 packets, 8075K bytes)
 pkts bytes target     prot opt in     out     source               destination         
    3   180 out-smtp   tcp  --  any    eth0    anywhere             anywhere             tcp dpt:smtp

Chain out-smtp (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     tcp  --  any    eth0    anywhere             anywhere             tcp dpt:smtpflags: RST/RST
    0     0 ACCEPT     tcp  --  any    eth0    anywhere             anywhere             tcp dpt:smtpflags: ACK/ACK
    3   180 LOG        tcp  --  any    eth0    anywhere             anywhere             tcp dpt:smtp ! owner GID match postfix LOG level warning uid
    3   180 DROP       tcp  --  any    eth0    anywhere             anywhere             tcp dpt:smtp ! owner GID match postfix
```

```
iptables -N out-smtp
iptables -A out-smtp -m tcp --tcp-flags RST RST -j ACCEPT
iptables -A out-smtp -m tcp --tcp-flags ACK ACK -j ACCEPT
iptables -A out-smtp -m owner ! --gid-owner postfix -j LOG --log-uid
iptables -A out-smtp -m owner ! --gid-owner postfix -j DROP

iptables -A OUTPUT -o eth0 -p tcp --dport 25 -j out-smtp
```
