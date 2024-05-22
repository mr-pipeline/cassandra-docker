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

-----
<b>T-Shooting</b>

```
cqlsh> DESCRIBE keyspaces;
```
```
CREATE KEYSPACE tahlilgaran WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};
```
```
ls /var/lib/cassandra/data/tahlilgaran
```
```
* in */etc/cassandra/cassandra.yaml* check:
```
data_file_directories:
    - /var/lib/cassandra/data
```
```
nodetool refresh tahlilgaran center
```
```
tail -f /var/log/cassandra/system.log
```
* Check Tables:
```
cqlsh > USE tahlilgaran;
DESCRIBE tables;
```
* Create Tables:
```
CREATE TABLE center (
    id UUID PRIMARY KEY,
    name text,
    description text
);
```
* Check Data Files:
```
/var/lib/cassandra/data/tahlilgaran/center-<UUID>
```
```
nodetool refresh
```
