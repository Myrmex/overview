---
layout: page
title: Backup
subtitle: Cadmus Deployment
---

To backup your Cadmus data, you must:

- backup its 3 **Mongo** databases (data, authentication, and log). You can leave out logs if you are not interested in keeping audit records.
- backup its **PostgreSQL** database(s) (indexes and optionally graph).

If you expose the database services from your Docker containers, as it is usually the case, you just have to use the corresponding database tools to dump databases. Just be sure to use the right port, as often ports are remapped when several services run in the same host machine.

For instance, here are typical dump commands for MongoDB and PostgreSQL:

```bash
mongodump -h 127.0.0.1 --port=27027 --archive=./backup/cadmus-PRJ-mongo.gz --gzip
psql -U postgres -d cadmus-PRJ-pgsql | gzip > ./backup/cadmus-PRJ-pgsql-${vardate}.gz
```

In Linux, you can just have a `.sh` batch file savinig your data somewhere, and then launch it periodically with [crontab](https://crontab.guru):

```sh
#!/bin/bash
# you can launch it by editing cron with crontab -e, using a line like this (daily dump at 3 AM):
# 00 03 * * * /home/crontab-scripts/cadmus-dump.sh

vardate=`date +%Y%m%d`
echo mongo all...
mongodump --port=27017 --archive=./backup/cadmus-PRJ-mongo-${vardate}.gz --gzip

echo mongo data...
mongodump --port=27017 --db cadmus-PRJ --archive=./backup/cadmus-PRJ-${vardate}.gz --gzip

echo mongo auth...
mongodump --port=27017 --db cadmus-PRJ-auth --archive=./backup/cadmus-PRJ-auth-${vardate}.gz --gzip

echo pgsql data...
psql -U postgres -d cadmus-PRJ-pgsql | gzip > ./backup/cadmus-PRJ-pgsql-${vardate}.gz

# if using MySql
# echo mysql...
#mysqldump --host 127.0.0.1 --port 3306 -uroot -pmysql cadmus-renovella | gzip > ./backup/cadmus-renovella-mysql-${vardate}.gz

echo completed!
```

Of course, you can then **transfer** your files somewhere else, e.g. (this script is placed in `backup` folder):

```bash
#!/bin/bash

vardate=`date +%Y%m%d`
echo upload
ftp-upload -h ftp.myserver.net -u user --password password *.gz
echo completed!
```
