## Ansible role to setup custom iptables rules

### How to use role

Just put role code in *roles/* directory and include it from playbook. This role doesn't set *Iptables Policy* to DROP but suggest to use REJECT it's considered more polite way. To make it working every host should have following variable set

````
---
host_iptables_rules:
  - { table: 'INPUT', cmd: "-m state --state RELATED,ESTABLISHED", action: 'ACCEPT', comment: 'Connection State'}
  - { table: 'INPUT', cmd: "-i lo", action: 'ACCEPT', comment: 'Allow lo'}
  - { table: 'INPUT', cmd: "-p icmp --icmp-type echo-request", action: 'ACCEPT', comment: 'Icmp echo-request'}
  - { table: 'INPUT', cmd: "-p icmp --icmp-type echo-reply", action: 'ACCEPT', comment: 'Icmp echo-reply'}
  - { table: 'INPUT', cmd: "-m tcp -p tcp --dport 22 -s 0.0.0.0/0 -m state --state NEW", action: 'ACCEPT', comment: 'SSH'}
  - { table: 'INPUT', cmd: "-m tcp -p tcp --dport 5666 -s 192.168.1.20/32 -m state --state NEW", action: 'ACCEPT', comment: 'NRPE MON'}
  - { table: 'INPUT', cmd: "", action: 'REJECT --reject-with="icmp-host-prohibited"', comment: 'REJECT everything else'}
  - { table: 'FORWARD', cmd: "", action: 'REJECT --reject-with="icmp-host-prohibited"', comment: 'REJECT everything else'}
````

with this rules for given host will be replaced by rules from _host_iptables_rules_
