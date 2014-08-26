## Ansible role to setup custom iptables rules

### How to use role

*!!! Warning !!!* This role will clear your iptables rules, so be careful.

Just put role code in *roles/* directory and include it from playbook. This role doesn't set *Iptables Policy* to DROP but suggest to use REJECT it's considered more polite way. To make it working every host should have following variable set

````yaml
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


It's possible to keep application specific rules in _host_iptables_rules_, too. But I prefer to use following code for it. Application
rules will not be added to iptables if anything with same comment is already there.


```yaml
- name: List Existing iptables rules
  sudo: yes
  always_run: yes
  command: iptables -L -n
  register: iptables_rules
  changed_when: iptables_rules.rc != 0

- name: Enable MYSQL traffic on test environment
  sudo: yes
  command: "iptables -I {{item.table}} {{item.cmd}} -j {{item.action}} -m comment --comment='{{item.comment}}'"
  with_items:
    - { table: 'INPUT', cmd: "-m tcp -p tcp --dport 3306 -s 192.168.255.0/24 -m state --state NEW", action: 'ACCEPT', comment: 'MYSQL VPN'}
    - { table: 'INPUT', cmd: "-m tcp -p tcp --dport 3306 -s 192.168.3.12/32 -m state --state NEW", action: 'ACCEPT', comment: 'MYSQL app'}
    - { table: 'INPUT', cmd: "-m tcp -p tcp --dport 3306 -s 192.168.1.17/32 -m state --state NEW", action: 'ACCEPT', comment: 'MYSQL apt'}
  when: environment == 'test' and iptables_rules.stdout.find('{{item.comment}}') == -1

- name: Enable MYSQL traffic on production systems
  sudo: yes
  command: "iptables -I {{item.table}} {{item.cmd}} -j {{item.action}} -m comment --comment='{{item.comment}}'"
  with_items:
    - { table: 'INPUT', cmd: "-m tcp -p tcp --dport 3306 -s 192.168.255.0/24 -m state --state NEW", action: 'ACCEPT', comment: 'MYSQL VPN'}
    - { table: 'INPUT', cmd: "-m tcp -p tcp --dport 3306 -s 192.168.3.12/32 -m state --state NEW", action: 'ACCEPT', comment: 'MYSQL app'}
  when: environment == 'prod' and iptables_rules.stdout.find('{{item.comment}}') == -1
```
