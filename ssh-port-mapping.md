---
title: ssh port mapping
date: 2018-04-26 15:06:41
tags: [ssh,port,map]
categories: network
---
# SSH port mapping
## SSH local port mapping
>**-L**  
[_bind_address_:]`port`:`host`:`hostport`
Specifies that the given `port` on the local (client) host is to be forwarded to the given `host` and `hostport` on the remote side. 
This works by allocating a socket to listen to `port` on the local side, optionally bound to the specified _bind_address_. Whenever a connection is made to this port, the connection is forwarded over the secure channel, and a connection is made to `host` port `hostport` from the remote machine. Port forwardings can also be specified in the configuration file. 
>
This is useful when you want to access a remote host within a LAN by jumping from a gateway.
`ssh -L port:host:hostport remote` will help you to map `host:hostport` to your local `port`. 
>


![enter image description here](https://github.com/losemyheaven/image-bank/raw/master/ssh/ssh-L.jpg)

### Usage:
A host M exports port 22 as its ssh service port, and its ip is 192.168.1.100.
A remote host R also run ssh service with 22 port, and it has two ip `192.168.1.200` and `10.67.1.200/24`
Local machine(10.67.1.1/24) could not directly access host M because they are in the different LAN.
After executing `ssh -L 1080:192.168.1.100:22  10.67.1.200`, you can use `ssh localhost:1080` to directly access ssh server running on host M.

<!--more--> 

 `ssh -L 1080:192.168.1.100:22  10.67.1.200` means:
 1. local client will listen 1080 port
 2. when data is sent from 9999 port to local 1080 port, it will be transfered to ssh server through ssh tunnel
 3. ssh server unpacks the data, and change the ip package header from source ip:port to 192.168.1.200:[new port] and then send the data package to 192.168.1.100:22. And also ssh server **must** record the [new port] is mapped to 10.67.1.1:9999. 
 4. some process will listen on 192.168.1.100:22, and when it receive data, it will reply to the source address 192.168.1.200:[new port].
 5. ssh server retransfer data received from [new port] to 10.67.1.1:9999
 
### Note that there is an address change in step 3, why such a change is needed? 
This is a similar tech in NAT(net address translation) wildly used in our routers. 
Because if source address is not changed, the process listening on 192.168.1.100:22 will directly reply to the source address 10.67.1.100:9999. However since there is not any rule to describe how to reach this address, it will fail.

## SSH remote port mapping
![enter image description here](https://github.com/losemyheaven/image-bank/raw/master/ssh/ssh-R.jpg)
>**-R**  
[_bind_address_:]`port`:`host`:`hostport`  
Specifies that the given `port` on the remote (server) host is to be forwarded to the given `host` and `hostport` on the local side. This works by allocating a socket to listen to  `port`  on the remote side, and whenever a connection is made to this port, the connection is forwarded over the secure channel, and a connection is made to  `host`  port  `hostport` from the local machine.
>By default, the listening socket on the server will be bound to the loopback interface only. This may be overridden by specifying a  _bind_address_. An empty  _bind_address_, or the address '*', indicates that the remote socket should listen on all interfaces. Specifying a remote  _bind_address_  will only succeed if the server's  **GatewayPorts**  option is enabled (see  **[sshd_config](https://linux.die.net/man/5/sshd_config)**(5)).
>If the  _port_  argument is '0', the listen port will be dynamically allocated on the server and reported to the client at run time.

Ha, more funny than ssh local port map.
They are quit similar, but work in different way.
Now ssh server will listen on both port 22 and `port` you specifie. on ssh server, `host`:`hostport` will be mapped to localhost:`port` on ssh server side.
Data sent to `port` will be received by ssh server and then be transfer through ssh tunnel. Then ssh client gets data and send to `host`:`hostport` with address translation mappings recorded as described above.

Have you ever considered that if you have a host with an public ip, and your local machine could connect it, how can you access local machine from the 'public' machine?
The answer is using ssh remote port mapping.

# SSH dynamic port mapping
![enter image description here](https://github.com/losemyheaven/image-bank/raw/master/ssh/ssh-D.jpg)
>**-D**  
[_ bind_address_:]_port_  
Specifies a local ''dynamic'' application-level port forwarding. This works by allocating a socket to listen to  _port_  on the local side, optionally bound to the specified  _bind_address_. Whenever a connection is made to this port, the connection is forwarded over the secure channel, and the application protocol is then used to determine where to connect to from the remote machine. Currently the SOCKS4 and SOCKS5 protocols are supported, and  **ssh**  will act as a SOCKS server. Only root can forward privileged ports. Dynamic port forwardings can also be specified in the configuration file.

ssh dynamic port mapping, often used to work around GFW throught your vpn tunnel.
Given that your local machine could connect through vpn to one of PCs in your working LAN, execute
```ssh -D 1080 remote_ip```
Now you have a SOCK proxy. Afterwards, each connecting attemp will be actually done by your remote PC which then returns data sent from internet website server to your local machine.
Here also an address translation mapping is involved.

# Reference
* https://linux.die.net/man/1/ssh
* https://www.cnblogs.com/eshizhan/archive/2012/07/16/2592902.html
* http://blog.creke.net/722.html

