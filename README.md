# TinyDNS-DJBDNS
TinyDNS/DJBDNS How to on Debian11
### What is djbdns
The djbdns software package is a DNS implementation created by Daniel J. Bernstein due to his frustrations with repeated BIND security holes.
### Installing dependencies
````
apt-get update
apt-get install build-essential
````
### Installing Daemintools
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

