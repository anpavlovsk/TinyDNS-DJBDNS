# TinyDNS-DJBDNS
TinyDNS/DJBDNS How to on Debian11
### What is djbdns
The djbdns software package is a DNS implementation created by Daniel J. Bernstein due to his frustrations with repeated BIND security holes.
### Installing dependencies
````
apt-get update
apt-get install build-essential
````
### Installing Daemontools
````
mkdir -p /package
chmod 1755 /package
cd /package
wget http://cr.yp.to/daemontools/daemontools-0.76.tar.gz
gunzip daemontools-0.76.tar
tar -xpf daemontools-0.76.tar
rm -f daemontools-0.76.tar
cd admin/daemontools-0.76
````
Compile and set up the daemontools programs:
````
package/install
cd compile
vi error.h
````
add in line 3 
````
#include errno.h
````
### Installing ucspi-tcp
tcpserver and tcpclient are easy-to-use command-line tools for building TCP client-server applications.
````
cd /usr/local/src
wget http://cr.yp.to/ucspi-tcp/ucspi-tcp-0.88.tar.gz
gunzip ucspi-tcp-0.88.tar
tar -xf ucspi-tcp-0.88.tar
cd ucspi-tcp-0.88
````
Compile the ucspi-tcp programs:
````
make
````
As root, install the ucspi-tcp programs under /usr/local:
````
make setup check
````
### Installing djbdns
````
mkdir packages
cd packages
wget //cr.yp.to/djbdns/djbdns-1.05.tar.gz 
tar -xzf djbdns-1.05.tar.gz
cd djbdns-1.05/ 
echo gcc -O2 -include /usr/include/errno.h > conf-cc 
make 
make setup check
````
### Running djbdns server
As root, create UNIX accounts named tinydns and dnslog.
````
/usr/sbin/useradd -s /bin/false tinydns
/usr/sbin/useradd -s /bin/false dnslog
```
As root, create an /etc/tinydns service directory configured with the IP address of the DNS server:
````
tinydns-conf tinydns dnslog /etc/tinydns 127.0.0.1
````
This directory contains logs and configuration files that you will change later.
The IP address must be configured on this computer. The IP address must not have a DNS cache or any other port-53 service. One computer can run a DNS server alongside a DNS cache as long as they are on separate IP addresses. The standard setup for small networks is to put a DNS cache on a private address such as 127.0.0.1 or 10.53.0.1, and a DNS server on a public address.

As root, tell svscan about the new service, and use svstat to check that the service is up:
````
ln -s /etc/tinydns /service
sleep 5
svstat /service/tinydns
````
