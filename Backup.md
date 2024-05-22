Get into cassandra container:
```
docker exec -it <container_name> /bin/bash
```
Using nodetool to take snapshot:
```
nodetool snapshot <keyspace_name>
```
Snapshots are saved in:
```
cd /var/lib/cassandra/data/<keyspace_name>
```
Exit of container:
```
exit
```
Copy snapshots from container to host:
```
docker cp <container_name>:/var/lib/cassandra/data/<keyspace_name>/snapshots/ <local_backup_directory>
```
