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
````
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
Output
````
root@debian11:/# svstat /service/tinydns
/service/tinydns: up (pid 2736) 202 seconds
````
### Running a DNS cache on a workstation
As root, create UNIX accounts named dnscache.
````
/usr/sbin/useradd -s /sbin/nologin -d /dev/null dnscache
````
As root, create an /etc/dnscache service directory:
````
dnscache-conf dnscache dnslog /etc/dnscache 192.168.1.16
````
This directory contains logs and a few configuration files that you can change later.
As root, tell svscan about the new service, and use svstat to check that the service is up:
````
ln -s /etc/dnscache /service
sleep 5
svstat /service/dnscache
````
Output
````
root@debian11:/# svstat /service/dnscache
/service/dnscache: up (pid 2737) 1255 seconds
````


Checking
````
ps fax
````
Output
````
2728 pts/0    S      0:00 /bin/sh /command/svscanboot
   2730 pts/0    S      0:00  \_ svscan /service
   2732 pts/0    S      0:00  |   \_ supervise tinydns
   2736 pts/0    S      0:00  |   |   \_ /usr/local/bin/tinydns
   2733 pts/0    S      0:00  |   \_ supervise log
   2738 pts/0    S      0:00  |   |   \_ multilog t ./main
   2734 pts/0    S      0:00  |   \_ supervise dnscache
   2737 pts/0    S      0:00  |   |   \_ /usr/local/bin/dnscache
   2735 pts/0    S      0:00  |   \_ supervise log
   2739 pts/0    S      0:00  |       \_ multilog t ./main
   2731 pts/0    S      0:00  \_ readproctitle service errors: .............................................................................................................................................
root@debian11:/# 
````
As root, put
````
nameserver 192.168.1.16
````
into /etc/resolv.conf, replacing any previous nameserver lines. You can skip this step if there are no nameserver lines or if /etc/resolv.conf doesn't exist.
````
cd /etc/dnscache/root/ip
touch 192.168
````
````
dig google.com
````
output
````
; <<>> DiG 9.16.27-Debian <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62292
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		276	IN	A	172.217.16.46

;; Query time: 327 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: Sun Aug 07 03:30:32 EDT 2022
;; MSG SIZE  rcvd: 55

````
Restart dnscache 
````
svc -t /service/dnscache
````
Output
````
root@debian11:~# dig google.com

; <<>> DiG 9.16.27-Debian <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 41745
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		119	IN	A	142.250.75.14

;; Query time: 0 msec
;; SERVER: 192.168.1.16#53(192.168.1.16)
;; WHEN: Sun Aug 07 04:06:25 EDT 2022
;; MSG SIZE  rcvd: 44

````
### Adding records to tynidns
````
cd /etc/service/tinydns/root/ 
./add-ns anpavlovsk.com 192.168.1.16 
./add-ns 1.168.192.in-addr.arpa 192.168.1.16
./add-host anpavlovsk.com 192.168.1.16 
./add-host mail.anpavlovsk.com 192.168.1.100 
make
````
Checking DNS records
````
dig anpavlovsk.com @127.0.0.1
````

Output
````
; <<>> DiG 9.16.27-Debian <<>> anpavlovsk.com @127.0.0.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 46900
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;anpavlovsk.com.			IN	A

;; ANSWER SECTION:
anpavlovsk.com.		86400	IN	A	192.168.1.16

;; AUTHORITY SECTION:
anpavlovsk.com.		259200	IN	NS	a.ns.anpavlovsk.com.

;; ADDITIONAL SECTION:
a.ns.anpavlovsk.com.	259200	IN	A	192.168.1.16

;; Query time: 3 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Sun Aug 07 04:22:16 EDT 2022
;; MSG SIZE  rcvd: 83

````
````
root@debian11:~# dig mail.anpavlovsk.com @127.0.0.1
````
Output
````
; <<>> DiG 9.16.27-Debian <<>> mail.anpavlovsk.com @127.0.0.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49053
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;mail.anpavlovsk.com.		IN	A

;; ANSWER SECTION:
mail.anpavlovsk.com.	86400	IN	A	192.168.1.100

;; AUTHORITY SECTION:
anpavlovsk.com.		259200	IN	NS	a.ns.anpavlovsk.com.

;; ADDITIONAL SECTION:
a.ns.anpavlovsk.com.	259200	IN	A	192.168.1.16

;; Query time: 3 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Sun Aug 07 04:26:09 EDT 2022
;; MSG SIZE  rcvd: 88
````
checking PTR record or reverse DNS
````
dig -x 192.168.1 @127.0.0.1
````
Output
````
; <<>> DiG 9.16.27-Debian <<>> -x 192.168.1 @127.0.0.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64038
;; flags: qr aa rd; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 0
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;1.168.192.in-addr.arpa.		IN	PTR

;; AUTHORITY SECTION:
1.168.192.in-addr.arpa.	2560	IN	SOA	a.ns.1.168.192.in-addr.arpa. hostmaster.1.168.192.in-addr.arpa. 1659268964 16384 2048 1048576 2560

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Sun Aug 07 04:59:08 EDT 2022
;; MSG SIZE  rcvd: 92
````
````
tinydns-get any anpavlovsk.com 127.0.0.1
````
Output
````
255 anpavlovsk.com:
130 bytes, 1+3+0+1 records, response, authoritative, noerror
query: 255 anpavlovsk.com
answer: anpavlovsk.com 2560 SOA a.ns.anpavlovsk.com hostmaster.anpavlovsk.com 1659268964 16384 2048 1048576 2560
answer: anpavlovsk.com 259200 NS a.ns.anpavlovsk.com
answer: anpavlovsk.com 86400 A 192.168.1.16
additional: a.ns.anpavlovsk.com 259200 A 192.168.1.16
````
````
tinydns-get ptr 1.168.192.in-addr.arpa 127.0.0.1
````
Output
````
12 1.168.192.in-addr.arpa:
92 bytes, 1+0+1+0 records, response, authoritative, noerror
query: 12 1.168.192.in-addr.arpa
authority: 1.168.192.in-addr.arpa 2560 SOA a.ns.1.168.192.in-addr.arpa hostmaster.1.168.192.in-addr.arpa 1659268964 16384 2048 1048576 2560
````
