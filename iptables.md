<center><h1>Iptables</h1></center>

-----
#What is this?
Iptables is just a shell interface to Netfilter which is the kernel module in charge of filtering network trafic. It is composed of tables named

- Filter
- Nat
- Mangle
- Raw
- Security

Each entry on each table is what one's call chains.

## Filter table
This table is primarly used as firewall. It is made to filter network packets and has 3 types of chains:

- INPUT which analyse incoming packets
- FORWARD manage bridges between interfaces
- OUTPUT analyse outcoming packets

Default policy can be affected to each entry chains

- DROP
- LOG
- ACCEPT
- SNAT
- DNAT
- REJECT

# Configuration
By default we want to block all the traffic then we allow case by case incoming or outcoming rules.

# Cheat-list
`iptables -L` list all rules of the table FILTER

     Chain INPUT (policy ACCEPT)
     target     prot opt source               destination         

     Chain FORWARD (policy ACCEPT)
     target     prot opt source               destination         

     Chain OUTPUT (policy ACCEPT)
     target     prot opt source               destination

`iptables -L -t<table>` to list another table
`iptables -F` flush iptables
`iptables -X` flush user chains

`iptables -L -v -n` List the rules with more details
##Allow traffic
From an already opened connection:

     iptables -A INPUT -m conntrack --ctstate ESTABLISHED -j ACCEPT
     iptables -A INPUT -p tcp -i eth0 --dport ssh -j ACCEPT

Loopback on localhost

     iptables -I INPUT 2 -i lo -j ACCEPT

ICMP

    iptables -A OUTPUT -p icmp -m conntrack --ctstate NEW,ESTABLISHED,RELATED -j ACCEPT
    iptables -A INPUT -p icmp -j ACCEPT
##Block traffic

     iptables -P INPUT DROP 

##Remove a rule

     iptables -L --line-numbers
     iptables -D OUTPUT 2

##Save all rules

    iptables-save -c

## Make rules persistent

    service iptables-persistent
