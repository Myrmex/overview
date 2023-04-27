---
layout: page
title: Backup
subtitle: Cadmus Deployment
---

To backup your Cadmus data, you must:

- backup its 3 Mongo databases (data, authentication, and log). You can leave out logs if you are not interested in keeping audit records.
- backup its single MySql database (used for indexes and graph).

If you expose the database services from your Docker containers, as it is usually the case, you just have to use the corresponding database tools to dump databases. Just be sure to use the right port, as often ports are remapped when several services run in the same host machine.

For instance, here are typical dump commands for MongoDB and MySql, using non-default ports:

```bash
mongodump --port=27127 --archive=./backup/cadmus-yourprj.gz --gzip > /dev/null 2>&1
mysqldump --host 127.0.0.1 --port 3326 -uroot -pmysql cadmus-yourprj > ./backup/cadmus-yourprj.sql
```

In Linux, you can just have a `.sh` batch file savinig your data somewhere, and then launch it periodically with [crontab](https://crontab.guru):

```sh
#!/bin/bash
# you can launch it by editing cron with crontab -e, using a line like this (daily dump at 3 AM):
# 00 03 * * * /home/crontab-scripts/cadmus-dump.sh

vardate=`date +%Y%m%d`

mongodump --port=27127 --archive=./backup/cadmus-yourprj-${vardate}.gz --gzip > /dev/null 2>&1
mysqldump --host 127.0.0.1 --port 3326 -uroot -pmysql cadmus-yourprj > ./backup/cadmus-yourprj-${vardate}.sql
```
