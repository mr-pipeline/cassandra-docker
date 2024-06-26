<b>Create Backup</b>

* Get into cassandra container:
```
docker exec -it <container_name> /bin/bash
```
* Using nodetool to take snapshot:
```
nodetool snapshot <keyspace_name>
```
* Snapshots are saved in:
```
cd /var/lib/cassandra/data/<keyspace_name>
```
* Exit of container:
```
exit
```
* Copy snapshots from container to host:
```
docker cp <container_name>:/var/lib/cassandra/data/<keyspace_name>/snapshots/ <local_backup_directory>
```

<b>Transfer Container to New VM</b>
```
docker stop <container_name>
docker commit <container_name> <image_name>
docker save -o <image_name>.tar <image_name>
```
```
scp <image_name>.tar user@new_vm:/path/to/destination
```

<b>Create Container in New VM</b>
```
docker load -i /path/to/destination/<image_name>.tar
```
```
docker run -d --name <new_container_name> <image_name>
```

<b>Restore Container in New VM</b>

* Copy Snapshots to New Container:
```
docker cp <local_backup_directory> <new_container_name>:/var/lib/cassandra/data/<keyspace_name>/snapshots/
```

* Enter to Container:
```
docker exec -it <new_container_name> /bin/bash
```

* Using nodetool to restore snapshots:
```
nodetool refresh <keyspace_name> <table_name>
```

* Exit Container:
```
exit
```


## T-Shooting

```
cqlsh> DESCRIBE keyspaces;
```
```
CREATE KEYSPACE tahlilgaran WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};
```
```
ls /var/lib/cassandra/data/<keyspace_name>
```

* in `/etc/cassandra/cassandra.yaml` check:
```
data_file_directories:
    - /var/lib/cassandra/data

listen_address: 192.168.1.100
rpc_address: 0.0.0.0
seed_provider:
  - class_name: org.apache.cassandra.locator.SimpleSeedProvider
    parameters:
      - seeds: "192.168.1.100,192.168.1.101"
```
```
nodetool refresh <keyspace_name> center
```
```
tail -f /var/log/cassandra/system.log
```
* Check Tables:
```
cqlsh > USE <keyspace_name>;
DESCRIBE tables;
```
* Create Tables:
```
CREATE TABLE <table_name> (
    id UUID PRIMARY KEY,
    name text,
    description text
);
```
* Check Nodes Status:
```
nodetool status
```
* Check Data Files:
```
/var/lib/cassandra/data/<keyspace_name>/<table_name>-<UUID>
```
```
nodetool refresh
```

-----

### Important Facts About Data Files:
get `UIID` from Log File:
```
mkdir -p /var/lib/cassandra/data/<keyspace_name>/<table_name>-<UUID>
```
```
chown -R cassandra:cassandra /var/lib/cassandra/data/<keyspace_name>
```
* Check SSTabls Files:
```
    *-Data.db
    *-Index.db
    *-Filter.db
    *-Summary.db
    *-TOC.txt
```
* Restart Cassandra Service:
```
sudo systemctl restart cassandra
```
```
sudo service cassandra restart
```
```
docker restart <container_id_or_name>
```
