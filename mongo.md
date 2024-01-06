## Restore a MongoDB collection from target database/cluster to destination database/cluster

To dump and restore a collection from a target MongoDB database/cluster to a destination database/cluster on the fly, without any intermediate dump file on disk, follow these steps (on Windows, adapt accordingly for Linux):

1. install mongodump and mongorestore utilities

Windows: https://www.mongodb.com/docs/database-tools/installation/installation-windows/
Linux: https://www.mongodb.com/docs/database-tools/installation/installation-linux/

2. create a folder to store the dump/restore script and its configuration files

```
mkdir C:\scripts\<your_folder>
```

3. restrict access permissions as much as possible on the new folder because it will contains some configuration files with the credentials required to authenticate against the source and destination Mongo databases/cluster

4. create a yml file for mongodump and fill it with the uri of the source MongoDB cluster and the password to access the source database/collection

```
uri: mongodb+srv://<source_mongodb_cluster_uri>
password: <password_of_user_with_access_permission_on_source_database_and_collection>
```

5. create a yml file for mongorestore and fill it with the uri of the destination MongoDB cluster and the password to access the destination database/collection. The format is the same of the previous file

6. run the following command or put it in a bat script (log is saved into a file called output.log):

```
mongodump --config=C:\scripts\<your_folder>\mongodump_config.yml --username <username> --db=<db_name> --collection <collection_name> --archive | mongorestore --config=C:\scripts\<your_folder>\mongorestore_config.yml --drop --archive --username <username> --nsFrom="<db_name>.<collection_name>" --nsTo="<db_name>.<collection_name>" 1> C:\scripts\<your_folder>\output.log 2>&1
```