### nsd:  zone master or slave , listen on 10053/udp. 
### unbound: cache forwarder, listen on 53/udp( default DNS  port)

### 1: Enable Service NSD and UNBOUND

```shell
 # rcctl enable nsd unbound
```

### 2: make nsd.conf  for running on port 10053 

/var/nsd/etc/nsd.conf 
```
server:
        hide-version: yes
        verbosity: 1
        database: "" # disable database
        port: 10053
remote-control:
        control-enable: yes
zone:
        name: "ace.local" 
        zonefile: "master/zone.ace.local"
```

### 3: put master zone file zone file /var/nsd/zones/master/ ...

/var/nsd/zones/master/zone.ace.local
``` 
$TTL 86400
@ SOA localhost. root.ace.local. (
                                2017082600; serial
                                28800; refresh
                                14400; retry
                                3600000; expire
                                86400;  minimum
                                )
ace.local.      IN      NS      ns.ace.local.
ns              IN      A       192.168.11.17
master          IN      CNAME   ns
pointer         IN      CNAME   ns
killer          IN      CNAME   ns
obsd8888        IN      CNAME   ns 
```

### 4: make unbound.conf for stub cache and forwarding 


/var/unbound/etc/unbound.conf
``` 
server:
        interface: 0.0.0.0
        #interface: 127.0.0.1@5353      # listen on alternative port
        interface: ::1
        access-control: 0.0.0.0/0 refuse
        access-control: 127.0.0.0/8 allow
        access-control: ::0/0 refuse
        access-control: ::1 allow
        do-not-query-localhost: no
        hide-identity: yes
        hide-version: yes
remote-control:
        control-enable: yes
        control-use-cert: no
        control-interface: /var/run/unbound.sock
stub-zone:
        name: "ace.local"
        stub-addr: 127.0.0.1@10053
forward-zone:
        name: "."
        forward-addr: 8.8.8.8
```

### 5: Start services

```shell
 # rcctl start nsd  unbound
```
