## Install Postgres 12 client on Amazon Linux EC2 instance

1. check if repo for postgres 12 is available

```
sudo  amazon-linux-extras | grep postgre
```

1. add repository

```
sudo tee /etc/yum.repos.d/pgdg.repo<<EOF
[pgdg12]
name=PostgreSQL 12 for RHEL/CentOS 7 - x86_64
baseurl=https://download.postgresql.org/pub/repos/yum/12/redhat/rhel-7-x86_64
enabled=1
gpgcheck=0
EOF
```

3. rebuild yum cache

```
sudo yum makecache
```

4. install Postgres 12 client

```
yum install postgresql12.x86_64
```

## Dump and restore a Posgres database in AWS RDS

* Original URL: https://medium.com/@arunshaji95/postgresql-pg-dump-and-pg-restore-ad8343f921bb
* Permanent copy: https://archive.ph/YY2R8
