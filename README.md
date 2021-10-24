# SONARQUBE WITH POSTGRES DATABASE STEPS BY STEPS

## `##++  Install & Setup Postgres Database for SonarQube  ++##`

```bash
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum install -y postgresql13-server
sudo /usr/pgsql-13/bin/postgresql-13-setup initdb
sudo systemctl enable postgresql-13
sudo systemctl start postgresql-13
systemctl status postgresql-13
```

#### Step 1: We need to check `` user which is created by default.
```bash
root@sonarqube:~ # cat /etc/passwd | grep postgres
postgres:x:26:26:PostgreSQL Server:/var/lib/pgsql:/bin/bash
root@sonarqube:~ #
```

#### Step 2: Set a password and connect to database (setting password as "admin" password).
```bash
root@sonarqube:~ # sudo passwd postgres
Changing password for user postgres.
New password:
BAD PASSWORD: The password is shorter than 8 characters
Retype new password:
passwd: all authentication tokens updated successfully.
root@sonarqube:~ #
```

#### Step 3: Switch user to `postgres` from root.
```bash
root@sonarqube:~ # su - postgres
Last login: Sat Oct 23 13:16:59 IST 2021 on pts/0
postgres@sonarqube:~ $
postgres@sonarqube:~ $ whoami
postgres
postgres@sonarqube:~ $
```

#### Step 4: Create a database user and database for sonarque.
```bash
postgres@sonarqube:~ $ createuser sonar
postgres@sonarqube:~ $ psql
psql (13.4)
Type "help" for help.

postgres=# ALTER USER sonar WITH ENCRYPTED PASSWORD 'admin';
ALTER ROLE
postgres=# CREATE DATABASE sonarqube OWNER sonar;
CREATE DATABASE
postgres=# GRANT ALL PRIVILEGES ON DATABASE sonarqube to sonar;
\GRANT
postgres=# \q
postgres@sonarqube:~ $ exit
root@sonarqube:~ #
```

#### Step 5: Restart postgres database to take latest changes effect.
```bash
root@sonarqube:~ # systemctl restart start postgresql-13
root@sonarqube:~ # 
root@sonarqube:~ # systemctl status postgresql-13
● postgresql-13.service - PostgreSQL 13 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-13.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2021-10-23 15:44:19 IST; 44min ago
     Docs: https://www.postgresql.org/docs/13/static/
 Main PID: 4867 (postmaster)
   CGroup: /system.slice/postgresql-13.service
           ├─4867 /usr/pgsql-13/bin/postmaster -D /var/lib/pgsql/13/data/
           ├─5025 postgres: logger
           ├─5124 postgres: checkpointer
           ├─5125 postgres: background writer
           ├─5126 postgres: walwriter
           ├─5127 postgres: autovacuum launcher
           ├─5128 postgres: stats collector
           └─5129 postgres: logical replication launcher

Oct 23 15:44:15 sonarqube systemd[1]: Starting PostgreSQL 13 database server...
Oct 23 15:44:18 sonarqube postmaster[4867]: 2021-10-23 15:44:18.708 IST [4867] LOG:  redirecting log output to logging coll...rocess
Oct 23 15:44:18 sonarqube postmaster[4867]: 2021-10-23 15:44:18.708 IST [4867] HINT:  Future log output will appear in dire..."log".
Oct 23 15:44:19 sonarqube systemd[1]: Started PostgreSQL 13 database server.
Hint: Some lines were ellipsized, use -l to show in full.
root@sonarqube:~ #
```

#### Step 6: We should see postgres is running on `5432`.

root@sonarqube:~ # netstat -plnt | grep 5432
tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN      11073/postmaster
tcp6       0      0 ::1:5432                :::*                    LISTEN      11073/postmaster
root@sonarqube:~ #


#### Step 7: Added below entries in `/etc/sysctl.conf`.
```bash
vm.max_map_count=524288
fs.file-max=131072
ulimit -n 131072
ulimit -u 8192
```

#### Step 8: Add below entries in `/etc/security/limits.conf`.
```bash
sonarqube   -   nofile   131072
sonarqube   -   nproc    8192
```

#### Step 9: Reboot the server.
```bash
init 6
```

## `##++++++++++++++++++++++  SonarQube Setup  ++++++++++++++++++++++##`

#### Step 1: Check Java version.
```bash
root@sonarqube:~ # java --version
-bash: java: command not found
root@sonarqube:~ #
```

#### Step 2: Install `java-11-openjdk`.
```bash
root@sonarqube:~ # yum install java-11-openjdk-devel
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirrors.piconets.webwerks.in
python-lxml.x86_64 0:3.2.1-4.el7                              ttmkfdir.x86_64 0:3.0.9-42.el7                   tzdata-java.noarch 0:2021c-1.el7
xorg-x11-font-utils.x86_64 1:7.5-21.el7                       xorg-x11-fonts-Type1.noarch 0:7.5-9.el7

Complete!
root@sonarqube:~ #

root@sonarqube:~ # java -version
openjdk version "11.0.12" 2021-07-20 LTS
OpenJDK Runtime Environment 18.9 (build 11.0.12+7-LTS)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.12+7-LTS, mixed mode, sharing)
root@sonarqube:~ #
```

#### Step 3: Download soarnqube zip in `/opt` dir.
Source: https://www.sonarqube.org/downloads/
```bash
root@sonarqube:~ # cd /opt/
root@sonarqube:opt #
root@sonarqube:opt # wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.2.46101.zip

--2021-10-23 16:33:10--  https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.2.46101.zip
Resolving binaries.sonarsource.com (binaries.sonarsource.com)... 91.134.125.245
Connecting to binaries.sonarsource.com (binaries.sonarsource.com)|91.134.125.245|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 276400757 (264M) [application/zip]
Saving to: ‘sonarqube-8.9.2.46101.zip’

100%[=========================================================================================>] 27,64,00,757 5.57MB/s   in 71s

2021-10-23 16:34:24 (3.71 MB/s) - ‘sonarqube-8.9.2.46101.zip’ saved [276400757/276400757]

root@sonarqube:opt #
root@sonarqube:opt # ll
total 269924
-rw-r--r--. 1 root root 276400757 Jul 27 13:25 sonarqube-8.9.2.46101.zip
root@sonarqube:opt #
```

#### Step 4: Unzip soarnqube zip and change folder name.
```bash
root@sonarqube:opt # pwd
/opt
root@sonarqube:opt #
root@sonarqube:opt # unzip sonarqube-8.9.2.46101.zip
root@sonarqube:opt #
root@sonarqube:opt # ll
total 269924
drwxr-xr-x. 11 root root       141 Jul 27 06:41 sonarqube-8.9.2.46101
-rw-r--r--.  1 root root 276400757 Jul 27 13:25 sonarqube-8.9.2.46101.zip
root@sonarqube:opt #
root@sonarqube:opt # mv sonarqube-8.9.2.46101 sonarqube
root@sonarqube:opt #
root@sonarqube:opt # ll
total 269924
drwxr-xr-x. 11 root root       141 Jul 27 06:41 sonarqube
-rw-r--r--.  1 root root 276400757 Jul 27 13:25 sonarqube-8.9.2.46101.zip
root@sonarqube:opt #
```

#### Step 5: Update sonar.properties with below information.

```bash
root@sonarqube:sonarqube-8.9.2.46101 # vim /opt/sonarqube/conf/sonar.properties

#sonar.jdbc.username=<sonar_database_username>
sonar.jdbc.username=sonar

#sonar.jdbc.password=<sonar_database_password>
sonar.jdbc.password=admin

#sonar.jdbc.url=jdbc:postgresql://<server_ip>/sonarqube?currentSchema=my_schema
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube

#sonar.search.javaOpts=-Xmx512m -Xms512m -XX:MaxDirectMemorySize=256m -XX:+HeapDumpOnOutOfMemoryError
sonar.search.javaOpts=-Xmx512m -Xms512m -XX:MaxDirectMemorySize=256m -XX:+HeapDumpOnOutOfMemoryError
```


#### Step 6: Create a `/etc/systemd/system/sonarqube.service` file start sonarqube service at the boot time.
```bash
cat >> /etc/systemd/system/sonarqube.service <<EOL
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
User=sonar
Group=sonar
PermissionsStartOnly=true
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start 
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
StandardOutput=syslog
LimitNOFILE=65536
LimitNPROC=4096
TimeoutStartSec=5
Restart=always

[Install]
WantedBy=multi-user.target
EOL
```
#### Step 7: Service file is created or not.
```bash
root@sonarqube:opt # ll /etc/systemd/system/sonarqube.service
-rw-r--r--. 1 root root 385 Oct 23 18:11 /etc/systemd/system/sonarqube.service
root@sonarqube:opt #
```

#### Step 8: Add sonar user to `/opt/sonarqube` directory.
```bash
root@sonarqube:sonarqube-8.9.2.46101 # useradd -d /opt/sonarqube sonar
useradd: warning: the home directory already exists.
Not copying any file from skel directory into it.
root@sonarqube:sonarqube-8.9.2.46101 #
root@sonarqube:sonarqube-8.9.2.46101 # cat /etc/passwd | grep sonar
sonar:x:1001:1001::/opt/sonarqube:/bin/bash
root@sonarqube:sonarqube-8.9.2.46101 #
```

#### Step 9: Grant ownership to `/opt/sonarqube` directory.
```bash
root@sonarqube:~ # cd
root@sonarqube:~ #
root@sonarqube:~ # pwd
/root
root@sonarqube:~ #
root@sonarqube:~ # ll /opt/sonarqube/
total 12
drwxr-xr-x. 6 root root   94 Jul 27 06:27 bin
drwxr-xr-x. 2 root root   50 Oct 23 16:55 conf
-rw-r--r--. 1 root root 7651 Jul 27 06:27 COPYING
drwxr-xr-x. 2 root root   24 Jul 27 06:27 data
drwxr-xr-x. 7 root root  132 Jul 27 06:27 elasticsearch
drwxr-xr-x. 4 root root   40 Jul 27 06:27 extensions
drwxr-xr-x. 6 root root  143 Jul 27 06:41 lib
drwxr-xr-x. 2 root root   24 Jul 27 06:27 logs
drwxr-xr-x. 2 root root   24 Jul 27 06:27 temp
drwxr-xr-x. 6 root root 4096 Jul 27 06:41 web
root@sonarqube:~ #
root@sonarqube:~ #
root@sonarqube:~ # chown -R sonar:sonar /opt/sonarqube
root@sonarqube:~ #
root@sonarqube:~ # ll /opt/sonarqube
total 12
drwxr-xr-x. 6 sonar sonar   94 Jul 27 06:27 bin
drwxr-xr-x. 2 sonar sonar   50 Oct 23 16:55 conf
-rw-r--r--. 1 sonar sonar 7651 Jul 27 06:27 COPYING
drwxr-xr-x. 2 sonar sonar   24 Jul 27 06:27 data
drwxr-xr-x. 7 sonar sonar  132 Jul 27 06:27 elasticsearch
drwxr-xr-x. 4 sonar sonar   40 Jul 27 06:27 extensions
drwxr-xr-x. 6 sonar sonar  143 Jul 27 06:41 lib
drwxr-xr-x. 2 sonar sonar   24 Jul 27 06:27 logs
drwxr-xr-x. 2 sonar sonar   24 Jul 27 06:27 temp
drwxr-xr-x. 6 sonar sonar 4096 Jul 27 06:41 web
root@sonarqube:~ #
```

#### Step 10: Open port `9000` from firewalld.
```bash
root@sonarqube:opt # firewall-cmd --add-port=9000/tcp --permanent
success
root@sonarqube:opt #
root@sonarqube:opt # firewall-cmd --reload
success
root@sonarqube:opt # firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources:
  services: ssh dhcpv6-client
  ports: 9000/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

root@sonarqube:opt #
```

#### Step 11: Reload the demon and start sonarqube service.
```bash
root@sonarqube:~ # systemctl daemon-reload
root@sonarqube:~ #
root@sonarqube:~ # systemctl start sonarqube.service
```

#### Step 12: Go to Browser and check GUI of Sonarqube.

http://<server_ip>:9000

```bash
root@sonarqube:opt # hostname -I
192.168.1.63
root@sonarqube:opt #
```

Eg:

http://192.168.1.63:9000

Default Credential
```bash
U: admin
P: admin
```


## `##++  Unable to access Sonarqube from browser?  ++##`

1. Make sure port 9000 is opened at security group leave
1. start sonar service as a sonar user
1. user correct database credentials in the sonar.properties
1. use instance which has atleast 2 GB of RAM

## `*************************   EOF   *************************`

https://linuxdevops.in/archives/category/devops