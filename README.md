# DC-9
Desarrollo del CTF DC:9

## 1. Download de la VM: 

https://www.vulnhub.com/entry/dc-9,412/

##2. Escaneo de Puertos

```
nmap -n -P0 -p- -sC -sV -O -T5 -oA full 10.10.10.138
Nmap scan report for 10.10.10.138
Host is up (0.00052s latency).
Not shown: 65533 closed ports
PORT   STATE    SERVICE VERSION
22/tcp filtered ssh
80/tcp open     http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Example.com - Staff Details - Welcome
MAC Address: 00:0C:29:F4:57:3F (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
```

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_1.jpg" width=80% />
