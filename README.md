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
