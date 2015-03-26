<center><h1>Iptables</h1></center>

-----
#What is this?
Iptables is just a shell interface to Netfilter which is the kernel module in charge of filtering network trafic. It is composed of tables named

- FILTER (Firewall) [ACCEPT, DENY, DROP, REJECT]
  - INPUT
  - FORWARD
  - OUTPUT
- NAT (Network address translation) [DNAT, SNAT, MASQUERADE]
  - PREROUTING
  - POSTROUTING
- Mangle (Packet modification on the fly) 
- Raw
- Security

Each entry on each table is what one's call chains.

## Filter table
This table is primarly used as firewall. It is made to filter network packets and has 3 types of chains:

- INPUT which analyse incoming packets
- FORWARD manage bridges between interfaces
- OUTPUT analyse outcoming packets

## NAT table

The targets are:

 - SNAT Allow to modify source address of a datagram
 - DNAT Allow to modify destination address
 - MASQUERADE Gateway
 - ACCEPT Accept a packet
 - DROP Reject a packet without notification
 - REJECT Reject a packet with deny notification
 - LOG Show the result on STDOUT (can be use with a prefix `--log-prefic '[...]'`)
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

## Commands

 `-A --append` Add a rule at the end
 `-D --delete` Remove a rule
 `-R --replace` Repleace a rule
 `-I --insert` Insert at a specific location *i.e.* `-I INPUT 1`
 `-L --list` Show rules
 `-F --flush` Empty all the rules of a chain *i.e.* `-F INPUT`
 `-N --new-chain` Create a new chain
 `-X --delete-chain` Remove a chain
 `-P --policy` Default polity for a chain (DENY, ACCEPT, REJECT, DROP)
 
 `-j` Action to take if match the rule (ACCEPT, LOG, DROP, ...)
 `-p --protocol` Which protocol (tcp, udp, icmp, all)
 `-s --source` Specify which addres
 `-d --destination` Destination address
 `-i --in-interface` Interface (eth0, lo)
 `-o --out-interface` 
 `-f --fragment` Fragmented packet
 `--sport --source-port` Source port
 `--dport --destination-port` Destination port
 `--icmp-type` Type of ICMP packet
 `--mac-source` MAC address
 `--state` (ESTABLISHED, NEW, INVALID, RELATED)
 `--to-destination` Used for DNAT to specify the destination address and port
 `--log-level` Verbosity of log
 `--log-prefix` Prefix of logs
 
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

## Forwarding
To allow traffic to be redirected through other network interfaces, `ip_forward` must be activated. 

    echo 1 > /proc/sys/net/ipv4/ip_forward
    
    
#Configuration Example

##Localhost loopback

    iptables -A INPUT -i lo -j ACCEPT
    iptables -A OUTPUT -o lo -j ACCEPT
    
##Log dropped

    iptables -P INPUT DROP
    iptables -P OUTPUT DROP
    iptables -P FORWARD DROP
    iptables -N LOG_DROP
    iptables -A LOG_DROP -j LOG --log-prefix '[IPTABLES DROP] : '
    iptables -A LOG_DROP -j DROP
    iptables -A FORWARD -j LOG_DROP
    iptables -A INPUT -j LOG_DROP
    iptables -A OUTPUT -j LOG_DROP
    
##DNS resolution

    iptables -A INPUT  -i eth0 --protocol udp --source-port 53 -j ACCEPT
    iptables -A OUTPUT -o eth0 --protocol udp --destination-port 53 -j ACCEPT
    iptables -A INPUT  -i eth0 --protocol tcp --destination-port 53 -j ACCEPT
    iptables -A OUTPUT -o eth0 --protocol tcp --destination-port 53 -j ACCEPT
    
##HTTP

    iptables -A INPUT  -i eth0 --protocol tcp --source-port 80 -m state --state ESTABLISHED -j LOG_ACCEPT
    iptables -A OUTPUT -i eth0 --protocol tcp --source-port 80 -m state --state NEW,ESTABLISHED -j LOG_ACCEPT
    iptables -A INPUT  -i eth0 --protocol tcp --source-port 443 -m state --state ESTABLISHED -j LOG_ACCEPT
    iptables -A OUTPUT -i eth0 --protocol tcp --source-port 443 -m state --state NEW,ESTABLISHED -j LOG_ACCEPT
    
##FTP

    iptables -A INPUT -i ppp0 -p tcp --sport 20 -m state --state ESTABLISHED,RELATED -j ACCEPT
    iptables -A OUTPUT -o ppp0 -p tcp --dport 20 -m state --state ESTABLISHED -j ACCEPT 
    iptables -A INPUT -i ppp0 -p tcp --sport 1024:65535 --dport 1024:65535 -m state --state ESTABLISHED -j ACCEPT
    iptables -A OUTPUT -o ppp0 -p tcp --sport 1024:65535 --dport 1024:65535 -m state --state ESTABLISHED,RELATED -j
    ACCEPT 
##Others

    SSH 22
    SMTP 25
    FTP 21
    
