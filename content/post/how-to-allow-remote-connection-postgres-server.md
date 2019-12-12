---
title: How to allow remote connection to Postgres server
author: ThaiNT
date: 2019-06-20 11:38:00+07:00
summary: This tutorial show you configurations on Postgres server which will allow connections from remote.
tags: ['database', 'ops']
---

## Step 1: Setup postgresql.conf

Check by command below:

```shell
netstat -nlt
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:11211         0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:3737          0.0.0.0:*               LISTEN
tcp6       0      0 :::22                   :::*                    LISTEN
```

You can see port `5432` (port of pg) is used by `127.0.0.1`. It means any
connection from outside to postgresql server will be refused.

Run this to file the config file of postgre:

```shell
sudo find / -name "postgresql.conf"
/etc/postgresql/9.5/main/postgresql.conf
```

Open file `postgresql.conf` by `vim /etc/postgresql/9.5/main/postgresql.conf`.
You can see the line:
```
listen_addresses = 'localhost'
```

Uncomment and change to:

```
listen_addresses = '*'
```

After saving your changes and restarting postgresql, run the first command
again:

```shell
netstat -nlt
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 127.0.0.1:11211         0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:5432            0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:2812          0.0.0.0:*               LISTEN
tcp6       0      0 ::1:11211               :::*                    LISTEN
tcp6       0      0 :::22                   :::*                    LISTEN
tcp6       0      0 :::5432                 :::*                    LISTEN
tcp6       0      0 ::1:25                  :::*                    LISTEN
```

You can see port `5432` is used by `0.0.0.0`.

## Step 2: Setup pg_hba.conf

Run this command to find the config file `pg_hba.conf`

```
sudo find / -name "pg_hba.conf"
/etc/postgresql/9.5/main/pg_hba.conf
```

Open file `pb_hba.conf` by `vim /etc/postgresql/9.5/main/pg_hba.conf`. Find and
change config like below:

```
host    all             all              0.0.0.0/0                       md5
host    all             all              ::/0                            md5
```

After restarting postgres, you can connect from client:

```
psql -h YOUR_IP_SERVER -U postgres
Password for user postgres:
psql (9.5.1, server 9.5)
Type "help" for help.

postgres=# \l
```
