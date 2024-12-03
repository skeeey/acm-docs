# Troubleshooting by using the database dump and restore 

In a production environment, back up your PostgreSQL database regularly as a database management task. The backup can also be used for debugging the multicluster global hub. 

## Symptom: Errors with multicluster global hub

You might experience various errors with multicluster global hub. You can use the database dump and restore for troubleshooting issues with multicluster global hub.

## Resolving the problem: Dumping the output of the database for dubugging

Sometimes you need to dump the output in the multicluster global hub database to debug a problem. The PostgreSQL database provides the `pg_dump` command line tool to dump the contents of the database. To dump data from localhost database server, run the following command:

```
pg_dump hoh > hoh.sql
```

To dump the multicluster global hub database located on a remote server with compressed format, use the command-line options to control the connection details, as shown in the following example:

```
pg_dump -h my.host.com -p 5432 -U postgres -F t hoh -f hoh-$(date +%d-%m-%y_%H-%M).tar
```

## Resolving the problem: Restore database from dump

To restore a PostgreSQL database, you can use the `psql` or `pg_restore` command line tools. The `psql` tool is used to restore plain text files created by `pg_dump`:

```
psql -h another.host.com -p 5432 -U postgres -d hoh < hoh.sql
```

The `pg_restore` tool is used to restore a PostgreSQL database from an archive created by `pg_dump` in one of the non-plain-text formats (custom, tar, or directory):

```
pg_restore -h another.host.com -p 5432 -U postgres -d hoh hoh-$(date +%d-%m-%y_%H-%M).tar
```
