---
layout: post
---

This pf(4) configuration mostly comes from [Tim's blog](http://blog.thechases.com/posts/bsd/aggressive-pf-config-for-ssh-protection/), and a bit of change in order to work on FreeBSD 13.1.

One more word for SSH protection, it's better to forbit Root login as well as Password login, this can be done in file */etc/ssh/sshd_config*.

```
cat /etc/pf.conf
##########
# Macros #
##########
# see also `sysctl net.inet.ip.portrange.{first,last}`
minefield="10000:65535"
ssh_alternate_port=11222


##########
# Tables #
##########
table <bruteforce> persist
table <troublemakers> persist
table <domesticv4> persist \
  file "/usr/local/share/geoip/ipv4_cn-aggregated.zone"
table <domesticv6> persist \
  file "/usr/local/share/geoip/ipv6_cn-aggregated.zone"


###########
# Options #
###########
set skip on lo


#########
# Rules #
#########
block return    # block stateless traffic
pass      # establish keep-state

# block brute-forcers
block quick proto tcp from <bruteforce> \
  to any port $ssh_alternate_port
block quick proto tcp from <troublemakers> \
  to any port $ssh_alternate_port

# non-domestic v4/v6 connections simply not allowed to touch SSH
# IPv4 GeoIP blocks: https://www.ipdeny.com/ipblocks/data/aggregated/cn-aggregated.zone
# IPv6 GeoIP blocks: https://www.ipdeny.com/ipv6/ipaddresses/aggregated/cn-aggregated.zone
#
block quick inet proto tcp \
  from ! <domesticv4> \
  to any port $ssh_alternate_port
block quick inet6 proto tcp \
  from ! <domesticv6> \
  to any port $ssh_alternate_port


# these made it through to the SSH port but abusing it
pass proto tcp from any to any \
  port {$ssh_alternate_port} \
  flags S/SA keep state \
  (max-src-conn 5, max-src-conn-rate 5/5, \
  overload <bruteforce> flush global \
  )

## these are just randomly probing

# port 22 is a dead-giveaway
# because we know we moved our SSH server elsewhere
# Also, we have no telnet or SMB
# so anyone poking there
# is up to no good
pass in proto tcp \
  from any \
  to any port { \
    telnet, \
    ssh, \
    pop3, \
    netbios-ns, \
    netbios-ssn, \
    microsoft-ds \
  } \
  synproxy state \
  tag trouble

# tag stuff in the range $minefield as trouble
pass in proto tcp from any to any port $minefield \
  synproxy state \
  tag trouble

# unless it's to our hole-in-one
pass in proto tcp from any to any port $ssh_alternate_port tag good

# if we're still trouble, add to the troublemakers
pass proto tcp from any to any port $ssh_alternate_port \
  tagged trouble \
  synproxy state \
  (max-src-conn 1, max-src-conn-rate 1/10, \
  overload <troublemakers> flush global \
  )

pass proto tcp from any to any port {http https} \
  keep state (max-src-conn 100, max-src-conn-rate 20/3)
```
