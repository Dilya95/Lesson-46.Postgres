# Домашнее задание 46: Postgres

## Задания
1. настроить hot_standby репликацию с использованием слотов<br>
2. настроить правильное резервное копирование<br>

## Структура

```text
.
├── Vagrantfile
├── postgresql.conf
├── pg_hba.conf
└── ansible
    ├── hosts
    ├── install_barman
    │   ├── defaults
    │   │   └── main.yml
    │   ├── tasks
    │   │   └── main.yml
    │   └── templates
    │       ├── barman.conf.j2
    │       └── node1.conf.j2
    ├── install_postgres
    │   ├── defaults
    │   │   └── main.yml
    │   └── tasks
    │       └── main.yml
    ├── postgres_replication
    │   ├── defaults
    │   │   └── main.yml
    │   ├── tasks
    │   │   └── main.yml
    │   └── templates
    │       ├── pg_hba.conf.j2
    │       └── postgresql.conf.j2
    └── provision.yml
```
## Выполнение

### 
```

```

### 
```

```

### 
```

```

### 
```

```


[root@otus-homework postgres]# vagrant ssh node1
vagrant@node1:~$ sudo -i

root@node1:~# apt-get update

root@node1:~# apt install postgresql postgresql-contrib -y

root@node1:~# systemctl start postgresql

root@node1:~# systemctl enable postgresql

Synchronizing state of postgresql.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable postgresql

root@node1:~# sudo -u postgres psql
could not change directory to "/root": Permission denied
psql (14.24 (Ubuntu 14.24-0ubuntu0.22.04.1))
Type "help" for help.

postgres=# CREATE USER replicator WITH REPLICATION Encrypted PASSWORD 'Otus2022!';
CREATE ROLE

postgres=# quit


root@node1:~# nano /etc/postgresql/16/main/postgresql.conf 

root@node1:~# grep -vE '^[[:space:]]*#' /etc/postgresql/16/main/postgresql.conf 

data_directory = '/var/lib/postgresql/14/main'    # use data in another directory
hba_file = '/etc/postgresql/14/main/pg_hba.conf'  # host-based authentication file
ident_file = '/etc/postgresql/14/main/pg_ident.conf'  # ident configuration file

external_pid_file = '/var/run/postgresql/14-main.pid'     # write an extra PID file

listen_addresses = 'localhost, 192.168.57.11'   # what IP address(es) to listen on;
port = 5432       # (change requires restart)
max_connections = 100     # (change requires restart)
unix_socket_directories = '/var/run/postgresql' # comma-separated list of directories

password_encryption = scram-sha-256 # scram-sha-256 or md5

ssl = on
ssl_cert_file = '/etc/ssl/certs/ssl-cert-snakeoil.pem'
ssl_key_file = '/etc/ssl/private/ssl-cert-snakeoil.key'

shared_buffers = 128MB      # min 128kB
dynamic_shared_memory_type = posix  # the default is the first option

wal_level = replica     # minimal, replica, or logical

max_wal_size = 1GB
min_wal_size = 80MB

max_wal_senders = 3   # max number of walsender processes
max_replication_slots = 3 # max number of replication slots

hot_standby = on      # "off" disallows queries during recovery
hot_standby_feedback = on   # send info from standby to prevent

log_directory = 'log'     # directory where log files are written,
log_filename = 'postgresql-%a.log'  # log file name pattern,
log_rotation_age = 1d     # Automatic rotation of logfiles will
log_rotation_size = 0   # Automatic rotation of logfiles will
log_truncate_on_rotation = on   # If on, an existing log file with the

log_line_prefix = '%m [%p]'   # special values:
log_timezone = 'UTC+3'

cluster_name = '14/main'      # added to process titles if nonempty

stats_temp_directory = '/var/run/postgresql/14-main.pg_stat_tmp'

datestyle = 'iso, mdy'
timezone = 'UTC+3'

lc_messages = 'en_US.UTF-8'     # locale for system error message
lc_monetary = 'en_US.UTF-8'     # locale for monetary formatting
lc_numeric = 'en_US.UTF-8'      # locale for number formatting
lc_time = 'en_US.UTF-8'       # locale for time formatting

default_text_search_config = 'pg_catalog.english'

include_dir = 'conf.d'      # include files ending in '.conf' from

root@node1:~# 

root@node1:~# nano /etc/postgresql/16/main/pg_hba.conf

root@node1:~# grep -vE '^[[:space:]]*#' /etc/postgresql/16/main/pg_hba.conf




local   all             postgres                                peer


local   all             all                                     peer
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            scram-sha-256
host    replication     all             ::1/128                 scram-sha-256
host    replication replicator    192.168.57.11/32        scram-sha-256
host    replication replicator    192.168.57.12/32        scram-sha-256


root@node1:~# systemctl restart postgresql

root@node1:~# sudo -u postgres psql
psql (16.15 (Ubuntu 16.15-0ubuntu0.24.04.1))
Type "help" for help.

postgres=# CREATE DATABASE otus_test;
CREATE DATABASE

postgres=# \l
                                                       List of databases
   Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | ICU Locale | ICU Rules |   Access privileges   
-----------+----------+----------+-----------------+-------------+-------------+------------+-----------+-----------------------
 otus_test | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | 
 postgres  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | 
 template0 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |             |             |            |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |             |             |            |           | postgres=CTc/postgres
(4 rows)

postgres=# select * from pg_stat_replication;

 pid  | usesysid |  usename   | application_name |  client_addr  | client_hostname | client_port |         backend_start         | backend_xmin |   state   | sent_lsn  | write_lsn | flush_lsn | replay_lsn | write_lag | flush_lag | replay_lag | sync_priority | sync_state |          reply_time           
------+----------+------------+------------------+---------------+-----------------+-------------+-------------------------------+--------------+-----------+-----------+-----------+-----------+------------+-----------+-----------+------------+---------------+------------+-------------------------------
 4851 |    16384 | replicator | 16/main          | 192.168.57.12 |                 |       43952 | 2026-10-29 05:20:00.683128-03 |              | streaming | 0/342D9D0 | 0/342D9D0 | 0/342D9D0 | 0/342D9D0  |           |           |            |             0 | async      | 2026-10-29 05:23:30.801277-03
(1 row)














[root@otus-homework postgres]# vagrant ssh node2
vagrant@node2:~$ sudo -i

root@node2:~# apt-get update

root@node2:~# apt install postgresql postgresql-contrib -y

root@node2:~# systemctl start postgresql

root@node2:~# systemctl enable postgresql

Synchronizing state of postgresql.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable postgresql


root@node2:~# systemctl stop postgresql

root@node2:~# rm -rf /var/lib/postgresql/16/main/*

root@node2:~# pg_basebackup -h 192.168.57.11 -U replicator -D /var/lib/postgresql/16/main -R -P
Password: 
23158/23158 kB (100%), 1/1 tablespace

root@node2:~# ls -la /var/lib/postgresql/16/main/
total 224
drwx------ 19 postgres postgres   4096 Oct 29 08:05 .
drwxr-xr-x  3 postgres postgres   4096 Oct 29 07:40 ..
-rw-------  1 root     root        225 Oct 29 08:05 backup_label
-rw-------  1 root     root     136894 Oct 29 08:05 backup_manifest
drwx------  5 root     root       4096 Oct 29 08:05 base
drwx------  2 root     root       4096 Oct 29 08:05 global
drwx------  2 root     root       4096 Oct 29 08:05 pg_commit_ts
drwx------  2 root     root       4096 Oct 29 08:05 pg_dynshmem
drwx------  4 root     root       4096 Oct 29 08:05 pg_logical
drwx------  4 root     root       4096 Oct 29 08:05 pg_multixact
drwx------  2 root     root       4096 Oct 29 08:05 pg_notify
drwx------  2 root     root       4096 Oct 29 08:05 pg_replslot
drwx------  2 root     root       4096 Oct 29 08:05 pg_serial
drwx------  2 root     root       4096 Oct 29 08:05 pg_snapshots
drwx------  2 root     root       4096 Oct 29 08:05 pg_stat
drwx------  2 root     root       4096 Oct 29 08:05 pg_stat_tmp
drwx------  2 root     root       4096 Oct 29 08:05 pg_subtrans
drwx------  2 root     root       4096 Oct 29 08:05 pg_tblspc
drwx------  2 root     root       4096 Oct 29 08:05 pg_twophase
-rw-------  1 root     root          3 Oct 29 08:05 PG_VERSION
drwx------  3 root     root       4096 Oct 29 08:05 pg_wal
drwx------  2 root     root       4096 Oct 29 08:05 pg_xact
-rw-------  1 root     root        398 Oct 29 08:05 postgresql.auto.conf
-rw-------  1 root     root          0 Oct 29 08:05 standby.signal

root@node2:~# chown -R postgres:postgres /var/lib/postgresql/16/main

root@node2:~# chmod 700 /var/lib/postgresql/16/main

root@node2:~#  ls -la /var/lib/postgresql/16/main
total 224
drwx------ 19 postgres postgres   4096 Oct 29 08:05 .
drwxr-xr-x  3 postgres postgres   4096 Oct 29 07:40 ..
-rw-------  1 postgres postgres    225 Oct 29 08:05 backup_label
-rw-------  1 postgres postgres 136894 Oct 29 08:05 backup_manifest
drwx------  5 postgres postgres   4096 Oct 29 08:05 base
drwx------  2 postgres postgres   4096 Oct 29 08:05 global
drwx------  2 postgres postgres   4096 Oct 29 08:05 pg_commit_ts
drwx------  2 postgres postgres   4096 Oct 29 08:05 pg_dynshmem
drwx------  4 postgres postgres   4096 Oct 29 08:05 pg_logical
drwx------  4 postgres postgres   4096 Oct 29 08:05 pg_multixact
drwx------  2 postgres postgres   4096 Oct 29 08:05 pg_notify
drwx------  2 postgres postgres   4096 Oct 29 08:05 pg_replslot
drwx------  2 postgres postgres   4096 Oct 29 08:05 pg_serial
drwx------  2 postgres postgres   4096 Oct 29 08:05 pg_snapshots
drwx------  2 postgres postgres   4096 Oct 29 08:05 pg_stat
drwx------  2 postgres postgres   4096 Oct 29 08:05 pg_stat_tmp
drwx------  2 postgres postgres   4096 Oct 29 08:05 pg_subtrans
drwx------  2 postgres postgres   4096 Oct 29 08:05 pg_tblspc
drwx------  2 postgres postgres   4096 Oct 29 08:05 pg_twophase
-rw-------  1 postgres postgres      3 Oct 29 08:05 PG_VERSION
drwx------  3 postgres postgres   4096 Oct 29 08:05 pg_wal
drwx------  2 postgres postgres   4096 Oct 29 08:05 pg_xact
-rw-------  1 postgres postgres    398 Oct 29 08:05 postgresql.auto.conf
-rw-------  1 postgres postgres      0 Oct 29 08:05 standby.signal




root@node2:~# nano /etc/postgresql/16/main/postgresql.conf

root@node2:~# grep -vE '^[[:space:]]*#' /etc/postgresql/16/main/postgresql.conf 

data_directory = '/var/lib/postgresql/16/main'    # use data in another directory
hba_file = '/etc/postgresql/16/main/pg_hba.conf'  # host-based authentication file
ident_file = '/etc/postgresql/16/main/pg_ident.conf'  # ident configuration file

external_pid_file = '/var/run/postgresql/16-main.pid'     # write an extra PID file

listen_addresses = 'localhost, 192.168.57.12'   # what IP address(es) to listen on;
port = 5432       # (change requires restart)
max_connections = 100     # (change requires restart)
unix_socket_directories = '/var/run/postgresql' # comma-separated list of directories

ssl = on
ssl_cert_file = '/etc/ssl/certs/ssl-cert-snakeoil.pem'
ssl_key_file = '/etc/ssl/private/ssl-cert-snakeoil.key'

shared_buffers = 128MB      # min 128kB
dynamic_shared_memory_type = posix  # the default is usually the first option

max_wal_size = 1GB
min_wal_size = 80MB

log_line_prefix = '%m [%p] %q%u@%d '    # special values:
log_timezone = 'Etc/UTC'

cluster_name = '16/main'      # added to process titles if nonempty

datestyle = 'iso, mdy'
timezone = 'Etc/UTC'

lc_messages = 'en_US.UTF-8'   # locale for system error message
lc_monetary = 'en_US.UTF-8'   # locale for monetary formatting
lc_numeric = 'en_US.UTF-8'    # locale for number formatting
lc_time = 'en_US.UTF-8'     # locale for time formatting

default_text_search_config = 'pg_catalog.english'
include_dir = 'conf.d'      # include files ending in '.conf' from
 

root@node2:~# systemctl start postgresql


root@node2:~# cat /var/lib/postgresql/16/main/postgresql.auto.conf
# Do not edit this file manually!
# It will be overwritten by the ALTER SYSTEM command.
primary_conninfo = 'user=replicator password=''Otus2022!'' channel_binding=prefer host=192.168.57.11 port=5432 sslmode=prefer sslcompression=0 sslcertmode=allow sslsni=1 ssl_min_protocol_version=TLSv1.2 gssencmode=prefer krbsrvname=postgres gssdelegation=0 target_session_attrs=any load_balance_hosts=disable'
root@node2:~# systemctl start postgresql@16-main
Job for postgresql@16-main.service failed because the service did not take the steps required by its unit configuration.
See "systemctl status postgresql@16-main.service" and "journalctl -xeu postgresql@16-main.service" for details.
root@node2:~# systemctl status postgresql@16-main
× postgresql@16-main.service - PostgreSQL Cluster 16-main
     Loaded: loaded (/usr/lib/systemd/system/postgresql@.service; enabled-runtime; preset: enabled)
     Active: failed (Result: protocol) since Thu 2026-10-29 08:16:38 UTC; 6s ago
   Duration: 12min 14.732s
    Process: 4748 ExecStart=/usr/bin/pg_ctlcluster --skip-systemctl-redirect 16-main start (code=exite>
        CPU: 60ms

Oct 29 08:16:38 node2 systemd[1]: Starting postgresql@16-main.service - PostgreSQL Cluster 16-main...
Oct 29 08:16:38 node2 postgresql@16-main[4748]: Error: /usr/lib/postgresql/16/bin/pg_ctl /usr/lib/post>
Oct 29 08:16:38 node2 postgresql@16-main[4748]: 2026-10-29 08:16:38.295 GMT [4753] LOG:  skipping miss>
Oct 29 08:16:38 node2 postgresql@16-main[4748]: 2026-10-29 08:16:38.295 UTC [4753] FATAL:  could not o>
Oct 29 08:16:38 node2 postgresql@16-main[4748]: pg_ctl: could not start server
Oct 29 08:16:38 node2 postgresql@16-main[4748]: Examine the log output.
Oct 29 08:16:38 node2 systemd[1]: postgresql@16-main.service: Can't open PID file /run/postgresql/16-m>
Oct 29 08:16:38 node2 systemd[1]: postgresql@16-main.service: Failed with result 'protocol'.
Oct 29 08:16:38 node2 systemd[1]: Failed to start postgresql@16-main.service - PostgreSQL Cluster 16-m>

root@node2:~# journalctl -u postgresql@16-main -n 50 --no-pager
Oct 29 07:40:30 node2 systemd[1]: Starting postgresql@16-main.service - PostgreSQL Cluster 16-main...
Oct 29 07:40:32 node2 systemd[1]: Started postgresql@16-main.service - PostgreSQL Cluster 16-main.
Oct 29 07:52:47 node2 systemd[1]: Stopping postgresql@16-main.service - PostgreSQL Cluster 16-main...
Oct 29 07:52:47 node2 systemd[1]: postgresql@16-main.service: Deactivated successfully.
Oct 29 07:52:47 node2 systemd[1]: Stopped postgresql@16-main.service - PostgreSQL Cluster 16-main.
Oct 29 08:09:19 node2 systemd[1]: Starting postgresql@16-main.service - PostgreSQL Cluster 16-main...
Oct 29 08:09:19 node2 postgresql@16-main[4712]: Error: /usr/lib/postgresql/16/bin/pg_ctl /usr/lib/postgresql/16/bin/pg_ctl start -D /var/lib/postgresql/16/main -l /var/log/postgresql/postgresql-16-main.log -s -o  -c config_file="/etc/postgresql/16/main/postgresql.conf"  exited with status 1:
Oct 29 08:09:19 node2 postgresql@16-main[4712]: 2026-10-29 08:09:19.644 GMT [4717] LOG:  skipping missing configuration file "/var/lib/postgresql/16/main/postgresql.auto.conf"
Oct 29 08:09:19 node2 postgresql@16-main[4712]: 2026-10-29 08:09:19.644 UTC [4717] FATAL:  could not open file "/var/lib/postgresql/16/main/PG_VERSION": Permission denied
Oct 29 08:09:19 node2 postgresql@16-main[4712]: pg_ctl: could not start server
Oct 29 08:09:19 node2 postgresql@16-main[4712]: Examine the log output.
Oct 29 08:09:19 node2 systemd[1]: postgresql@16-main.service: Can't open PID file /run/postgresql/16-main.pid (yet?) after start: No such file or directory
Oct 29 08:09:19 node2 systemd[1]: postgresql@16-main.service: Failed with result 'protocol'.
Oct 29 08:09:19 node2 systemd[1]: Failed to start postgresql@16-main.service - PostgreSQL Cluster 16-main.


root@node2:~# pg_lsclusters
Ver Cluster Port Status        Owner    Data directory              Log file
16  main    5432 down,recovery postgres /var/lib/postgresql/16/main /var/log/postgresql/postgresql-16-main.log






root@node2:~# systemctl start postgresql@16-main
root@node2:~# systemctl status postgresql@16-main
● postgresql@16-main.service - PostgreSQL Cluster 16-main
     Loaded: loaded (/usr/lib/systemd/system/postgresql@.service; enabled-runtime; preset: enabled)
     Active: active (running) since Thu 2026-10-29 08:20:02 UTC; 4s ago
    Process: 4770 ExecStart=/usr/bin/pg_ctlcluster --skip-systemctl-redirect 16-main start (code=exite>
   Main PID: 4779 (postgres)
      Tasks: 5 (limit: 852)
     Memory: 52.9M (peak: 61.6M)
        CPU: 206ms
     CGroup: /system.slice/system-postgresql.slice/postgresql@16-main.service
             ├─4779 /usr/lib/postgresql/16/bin/postgres -D /var/lib/postgresql/16/main -c config_file=>
             ├─4780 "postgres: 16/main: checkpointer "
             ├─4781 "postgres: 16/main: background writer "
             ├─4782 "postgres: 16/main: startup recovering 000000010000000000000003"
             └─4783 "postgres: 16/main: walreceiver streaming 0/342D9D0"

Oct 29 08:20:00 node2 systemd[1]: Starting postgresql@16-main.service - PostgreSQL Cluster 16-main...
Oct 29 08:20:02 node2 systemd[1]: Started postgresql@16-main.service - PostgreSQL Cluster 16-main.
root@node2:~# pg_lsclusters
Ver Cluster Port Status          Owner    Data directory              Log file
16  main    5432 online,recovery postgres /var/lib/postgresql/16/main /var/log/postgresql/postgresql-16-main.log

root@node2:~# sudo -u postgres psql
psql (16.15 (Ubuntu 16.15-0ubuntu0.24.04.1))
Type "help" for help.

postgres=# \l
                                                       List of databases
   Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | ICU Locale | ICU Rules |   Access privileges   
-----------+----------+----------+-----------------+-------------+-------------+------------+-----------+-----------------------
 otus_test | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | 
 postgres  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | 
 template0 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |             |             |            |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |             |             |            |           | postgres=CTc/postgres
(4 rows)



postgres=# select * from pg_stat_wal_receiver;

 pid  |  status   | receive_start_lsn | receive_start_tli | written_lsn | flushed_lsn | received_tli |      last_msg_send_time       |    last_msg_receipt_time     | latest_end_lsn |        latest_end_time        | slot_name |  sender_host  | sender_port |                                                                                                                                                                     conninfo                                                                                                                                                                     
------+-----------+-------------------+-------------------+-------------+-------------+--------------+-------------------------------+------------------------------+----------------+-------------------------------+-----------+---------------+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 4783 | streaming | 0/3000000         |                 1 | 0/342D9D0   | 0/342D9D0   |            1 | 2026-10-29 08:24:30.856588+00 | 2026-10-29 08:24:30.83335+00 | 0/342D9D0      | 2026-10-29 08:20:00.693088+00 |           | 192.168.57.11 |        5432 | user=replicator password=******** channel_binding=prefer dbname=replication host=192.168.57.11 port=5432 fallback_application_name=16/main sslmode=prefer sslcompression=0 sslcertmode=allow sslsni=1 ssl_min_protocol_version=TLSv1.2 gssencmode=prefer krbsrvname=postgres gssdelegation=0 target_session_attrs=any load_balance_hosts=disable
(1 row)












### Настройка резервного копирования

## На хосте node1
```
root@node1:~# apt install barman-cli -y

root@node1:~# su postgres

postgres@node1:/root$ cd

postgres@node1:~$ pwd
/var/lib/postgresql

postgres@node1:~$ ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/var/lib/postgresql/.ssh/id_rsa): 
Created directory '/var/lib/postgresql/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /var/lib/postgresql/.ssh/id_rsa
Your public key has been saved in /var/lib/postgresql/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:phOzlSC/l3YOfjRku4RDqk7lWSQGbIkvHJni+0ok1v0 postgres@node1
The key's randomart image is:
+---[RSA 4096]----+
|   =..           |
|. = +.           |
|.o +. + .        |
| .+ o+ +..o      |
|.o.o .=oS+ .     |
|+.   oo@o.=      |
| .. ..BE=+.o     |
|.  o.  = +o      |
| ....   ...      |
+----[SHA256]-----+

postgres@node1:~$ cat ~/.ssh/id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCtuBdZVQMyJPuzaZXUMPAkIMnbZIGJh491cZbVQ3wu6gGBVv7aU1rJR2LYEnjGGUITDPAphZCc+JD1uahE9+5aJpSDJa9x/m6Qr6F0DFs3nmVukzMW/5hBitQHFDBy6dcG935X8CyYcDP6C6S1KoxpEW/91PLYj8K+oihlF8LxxFpWESi08yTl/SZc0L+LHwpBL9B3m95NzSNciT+kjUex0PRTeDoHce6/U14vs2Ix1ALoKPb2yGt/ENzC8W5Gas0mP+LS5gUznypkpMp07DNRtbcwW92ia/aT+EkFupsaJbflRGcbsHmW42u5H0EvTHwxY3rfiFtnBTDciUXSs6Ll9EvtJUaTqPHWpwh5gBUR4ED39esPykyc2gvHb7DTDs7VdKAjHyMCurrDn4lUZfUAbhHW70FXy594h/+MylCQQC67m77UvstvFyg+8VepKrEHP5F2VGEQlHZt5JCTBIVEfdgM47Iku0Z9sWdxJbvXQrnqHD96iggr+NveT0QHu95/38Z6YzzOWD9ArpBo4iISKB9H7AV4oBlU/xyt6oD4i9JPEqUjqKgwj1vePcL6TNYps7kaLN6Ep3N8VXJwabgixVYn9oHIEwshF6dHveozqwJL5sZ3X2jl0APgYJwop5wgoQj6/iXucucTNvCi1ZUCwNcdpgJlrmSFwvOOEYGYAQ== postgres@node1

root@node1:~# nano /var/lib/postgresql/.ssh/authorized_keys
root@node1:~# chown postgres:postgres /var/lib/postgresql/.ssh/authorized_keys
chmod 600 /var/lib/postgresql/.ssh/authorized_keys
root@node1:~# ls -la /var/lib/postgresql/.ssh
total 20
drwx------ 2 postgres postgres 4096 Oct 29 10:09 .
drwxr-xr-x 4 postgres postgres 4096 Oct 29 10:03 ..
-rw------- 1 postgres postgres  739 Oct 29 10:09 authorized_keys
-rw------- 1 postgres postgres 3381 Oct 29 08:55 id_rsa
-rw-r--r-- 1 postgres postgres  740 Oct 29 08:55 id_rsa.pub



root@node1:~# sudo -u postgres psql
psql (16.15 (Ubuntu 16.15-0ubuntu0.24.04.1))
Type "help" for help.

postgres=# CREATE USER barman WITH REPLICATION Encrypted PASSWORD 'Otus2022!';
CREATE ROLE

postgres=# quit


root@node1:~# grep -vE '^[[:space:]]*#' /etc/postgresql/16/main/pg_hba.conf
# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
# IPv6 local connections:
host    all             all             ::1/128                 scram-sha-256
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            scram-sha-256
host    replication     all             ::1/128                 scram-sha-256
host    replication replicator    192.168.57.11/32        scram-sha-256
host    replication replicator    192.168.57.12/32        scram-sha-256
host    all             barman       192.168.57.13/32        scram-sha-256
host    replication barman       192.168.57.13/32      scram-sha-256

root@node1:~# systemctl restart postgresql

root@node1:~#  sudo -u postgres psql
psql (16.15 (Ubuntu 16.15-0ubuntu0.24.04.1))
Type "help" for help.

postgres=# CREATE DATABASE otus;
CREATE DATABASE

postgres=# \c otus;
You are now connected to database "otus" as user "postgres".

otus=# CREATE TABLE test (id int, name varchar(30));
CREATE TABLE

otus=# INSERT INTO test VALUES (1, 'alex'); 
INSERT 0 1

root@node1:~# sudo -u postgres psql
psql (16.15 (Ubuntu 16.15-0ubuntu0.24.04.1))
Type "help" for help.

postgres=# GRANT EXECUTE ON FUNCTION pg_backup_start(text, boolean) TO barman;
GRANT EXECUTE ON FUNCTION pg_backup_stop(boolean) TO barman;
GRANT EXECUTE ON FUNCTION pg_switch_wal() TO barman;
GRANT EXECUTE ON FUNCTION pg_create_restore_point(text) TO barman;

GRANT pg_read_all_settings TO barman;
GRANT pg_read_all_stats TO barman;
GRANT pg_checkpoint TO barman;
GRANT
GRANT
GRANT
GRANT
GRANT ROLE
GRANT ROLE
GRANT ROLE
postgres=# \du barman
      List of roles
 Role name | Attributes  
-----------+-------------
 barman    | Replication

postgres=# 

```




## На хосте barman
```
vagrant@barman:~$ sudo -i

root@barman:~# apt-get update

root@barman:~# apt install barman-cli barman postgresql -y

root@barman:~# su barman

barman@barman:/root$ cd

barman@barman:~$ pwd
/var/lib/barman

barman@barman:~$ mkdir .ssh

barman@barman:~$ nano /var/lib/barman/.ssh/authorized_keys

barman@barman:~/.ssh$ ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/var/lib/barman/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /var/lib/barman/.ssh/id_rsa
Your public key has been saved in /var/lib/barman/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:OyWKYsvaOXhbyPFH9CF6169MsKZcdcsN5TTd8Sf50bI barman@barman
The key's randomart image is:
+---[RSA 4096]----+
|               . |
|               o=|
|      o .     B.*|
|     o o o   + *o|
|  . . o S + o E .|
| . + + o B + +   |
| .= + o * . + .  |
|.+o= o + + .     |
|.o*o  o   o      |
+----[SHA256]-----+
barman@barman:~/.ssh$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDkA08ooKcSVHMDFOioi+zg9Bwz/0dk9hyKPwRCfeoM0dptMweF+Ewj8gbn9djyADopBFYV/Ihwv5PD96DfbYBfMWrIdjRdXetkqL96E0At8glWfClb0/FCuxdgZBOyyFZtS3IbRDFg5Jh/LKy9l92yB8+iUeN2Knv47IMkqLau/iczGgmpWiMSqZojcKO9QqM4/3NYAWvOHNLcQgp5nFaOHyRL0m3Y0kMFhFyJioRyjlwhEjaVSTAjv11WjikLLV1eHXhN8RojrK+tQ1letywwycat0o54Fr5gcwzBAUDe84xzuuGtMq+LQSAb9MjMfuXGgFoM/2tvKqVPe6y8j713hNhOCjVEQNz9AGnTGcQ4kOC1Rr1NiPSaBm9SD4WCazA8tb7IAPVFu3W1aKfHnhTxWnO2YerWtJMsvy2mpEb4hjCC6/U6jwuwA9/XyLJPAcw0Mb5SaQGUswnVgOPeqLPaFa8Y/nIy49j7F7VaYsq29ImyTPA2fEr9TztYDWV8jnD7DSoqAT3/9NlAv3/LiKLU4CPPD4MOSwB0bAhViwpwX+uY8v95RYfTunnmUflM54tWi8n02OcIk5J1uhNbU4nsvp6ypQZeJUzlDbiQlYJ+3XvtQV7KFf/JCTqChklYewQDPNqjqCXJtj75KRPD23PUKa17ChcGqo52WoduNdaH0w== barman@barman

barman@barman:~$ nano ~/.pgpass

barman@barman:~$ ls -la
total 24
drwxr-x---  4 barman barman 4096 Oct 29 09:34 .
drwxr-xr-x 43 root   root   4096 Oct 29 08:41 ..
-rw-rw-r--  1 barman barman    2 Oct 29 09:34 cfg_changes.queue
drwxrwxr-x  3 barman barman 4096 Oct 29 08:55 .local
-rw-rw-r--  1 barman barman   38 Oct 29 09:07 .pgpass
drwxrwxr-x  2 barman barman 4096 Oct 29 08:56 .ssh

barman@barman:~$ cat ~/.pgpass
192.168.57.11:5432:*:barman:Otus2022!

barman@barman:~$ chmod 600 .pgpass

barman@barman:~$ psql -h 192.168.57.11 -U barman -d postgres 
psql (16.15 (Ubuntu 16.15-0ubuntu0.24.04.1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.
postgres=> \q

barman@barman:~$ psql -h 192.168.57.11 -U barman -c "IDENTIFY_SYSTEM" replication=1
      systemid       | timeline |  xlogpos  | dbname 
---------------------+----------+-----------+--------
 7701991482240921098 |        1 | 0/385D650 | 
(1 row)


root@barman:~# grep -vE '^[[:space:]]*;' /etc/barman.conf

[barman]
barman_user = barman
backup_method = rsync
archiver = on

configuration_files_directory = /etc/barman.d

barman_home = /var/lib/barman

log_file = /var/log/barman/barman.log

log_level = INFO

compression = gzip

immediate_checkpoint = true

last_backup_maximum_age = 4 DAYS

minimum_redundancy = 1

retention_policy = REDUNDANCY 3


root@barman:~# chown -R barman:barman /etc/barman.conf

root@barman:~# nano /etc/barman.d/node1.conf

root@barman:~# chown -R barman:barman /etc/barman.d/node1.conf 

root@barman:~# cat /etc/barman.d/node1.conf
[node1]
                    #Описание задания
description = "backup node1"
#Команда подключения к хосту node1
ssh_command = ssh postgres@192.168.57.11 
#Команда для подключения к postgres-серверу
conninfo = host=192.168.57.11 user=barman port=5432 dbname=postgres
retention_policy_mode = auto
retention_policy = RECOVERY WINDOW OF 7 days
wal_retention_policy = main
streaming_archiver=on
#Указание префикса, который будет использоваться как $PATH на хосте node1
path_prefix = /usr/pgsql-14/bin/
#настройки слота
create_slot = auto
slot_name = node1
#Команда для потоковой передачи от postgres-сервера
streaming_conninfo = host=192.168.57.11 user=barman 
#Тип выполняемого бекапа
backup_method = postgres
archiver = off


root@barman:~# su - barman

barman@barman:~$ barman switch-wal node1
The WAL file 000000010000000000000003 has been closed on server 'node1'

barman@barman:~$ barman cron
Starting WAL archiving for server node1

barman@barman:~$ barman check node1
Server node1:
  PostgreSQL: OK
  superuser or standard user with backup privileges: OK
  PostgreSQL streaming: OK
  wal_level: OK
  replication slot: OK
  directories: OK
  retention policy settings: OK
  backup maximum age: FAILED (interval provided: 4 days, latest backup age: No available backups)
  backup minimum size: OK (0 B)
  wal maximum age: OK (no last_wal_maximum_age provided)
  wal size: OK (0 B)
  compression settings: OK
  failed backups: OK (there are 0 failed backups)
  minimum redundancy requirements: FAILED (have 0 backups, expected at least 1)
  pg_basebackup: OK
  pg_basebackup compatible: OK
  pg_basebackup supports tablespaces mapping: OK
  systemid coherence: OK (no system Id stored on disk)
  pg_receivexlog: OK
  pg_receivexlog compatible: OK
  receive-wal running: OK
  archiver errors: OK


barman@barman:~$ barman backup node1
Starting backup using postgres method for server node1 in /var/lib/barman/node1/base/20261029T095609
Backup start at LSN: 0/4000060 (000000010000000000000004, 00000060)
Starting backup copy via pg_basebackup for 20261029T095609
WARNING: pg_basebackup does not copy the PostgreSQL configuration files that reside outside PGDATA. Please manually backup the following files:
  /etc/postgresql/16/main/postgresql.conf
  /etc/postgresql/16/main/pg_hba.conf
  /etc/postgresql/16/main/pg_ident.conf

Copy done (time: less than one second)
Finalising the backup.
This is the first backup for server node1
WAL segments preceding the current backup have been found:
  000000010000000000000003 from server node1 has been removed
Backup size: 36.8 MiB
Backup end at LSN: 0/6000000 (000000010000000000000005, 00000000)
Backup completed (start time: 2026-10-29 09:56:09.391436, elapsed time: less than one second)
Processing xlog segments from streaming for node1
  000000010000000000000004
  000000010000000000000005
barman@barman:~$ 
```




### Проверка восстановления из бекапов
## Дропаем БД в node1
```
root@node1:~# sudo -u postgres psql
psql (16.15 (Ubuntu 16.15-0ubuntu0.24.04.1))
Type "help" for help.

postgres=# \l
                                                       List of databases
   Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | ICU Locale | ICU Rules |   Access privileges   
-----------+----------+----------+-----------------+-------------+-------------+------------+-----------+-----------------------
 otus      | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | 
 otus_test | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | 
 postgres  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | 
 template0 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |             |             |            |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |             |             |            |           | postgres=CTc/postgres
(5 rows)

postgres=# DROP DATABASE otus;
DROP DATABASE

postgres=# DROP DATABASE otus_test; 
DROP DATABASE

postgres=# \l
                                                       List of databases
   Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | ICU Locale | ICU Rules |   Access privileges   
-----------+----------+----------+-----------------+-------------+-------------+------------+-----------+-----------------------
 postgres  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | 
 template0 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |             |             |            |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |             |             |            |           | postgres=CTc/postgres
(3 rows)
```

## Запускаем восстановление в barman
```
barman@barman:~$ barman recover node1 20261029T095609 /var/lib/postgresql/16/main/ --remote-ssh-comman "ssh postgres@192.168.57.11"
Starting remote restore for server node1 using backup 20261029T095609
Destination directory: /var/lib/postgresql/16/main/
Remote command: ssh postgres@192.168.57.11
Copying the base backup.
Copying required WAL segments.
Generating archive status files
Identify dangerous settings in destination directory.

WARNING
The following configuration files have not been saved during backup, hence they have not been restored.
You need to manually restore them in order to start the recovered PostgreSQL instance:

    postgresql.conf
    pg_hba.conf
    pg_ident.conf

Recovery completed (start time: 2026-10-29 10:13:01.715502+00:00, elapsed time: 3 seconds)
Your PostgreSQL server has been successfully prepared for recovery!
barman@barman:~$ 
```



## Делаем рестарт сервиса и проверяем восстановление

```
root@node1:~#  systemctl restart postgresql@16-main

root@node1:~#  systemctl status postgresql@16-main
● postgresql@16-main.service - PostgreSQL Cluster 16-main
     Loaded: loaded (/usr/lib/systemd/system/postgresql@.service; enabled-runtime; preset: enabled)
     Active: active (running) since Thu 2026-10-29 10:14:57 UTC; 13s ago
    Process: 5994 ExecStart=/usr/bin/pg_ctlcluster --skip-systemctl-redirect 16-main start (code=exited, >
   Main PID: 5999 (postgres)
      Tasks: 8 (limit: 852)
     Memory: 39.1M (peak: 43.3M)
        CPU: 234ms
     CGroup: /system.slice/system-postgresql.slice/postgresql@16-main.service
             ├─5999 /usr/lib/postgresql/16/bin/postgres -D /var/lib/postgresql/16/main -c config_file=/et>
             ├─6000 "postgres: 16/main: checkpointer "
             ├─6001 "postgres: 16/main: background writer "
             ├─6003 "postgres: 16/main: walwriter "
             ├─6004 "postgres: 16/main: autovacuum launcher "
             ├─6005 "postgres: 16/main: logical replication launcher "
             ├─6017 "postgres: 16/main: walsender replicator 192.168.57.12(35382) streaming 0/60000D8"
             └─6023 "postgres: 16/main: walsender barman 192.168.57.13(58372) streaming 0/60000D8"

Oct 29 10:14:55 node1 systemd[1]: Starting postgresql@16-main.service - PostgreSQL Cluster 16-main...
Oct 29 10:14:57 node1 systemd[1]: Started postgresql@16-main.service - PostgreSQL Cluster 16-main.


root@node1:~# sudo -u postgres psql
psql (16.15 (Ubuntu 16.15-0ubuntu0.24.04.1))
Type "help" for help.

postgres=# \l
                                                       List of databases
   Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | ICU Locale | ICU Rules |   Access privileges   
-----------+----------+----------+-----------------+-------------+-------------+------------+-----------+-----------------------
 otus      | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | 
 otus_test | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | 
 postgres  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | 
 template0 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |             |             |            |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |             |             |            |           | postgres=CTc/postgres
(5 rows)

```


