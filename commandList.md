# Verify that a Cassandra 3.x node is running

The script running in the background is installing JDK 8 and Cassandra 3.x. Once Cassandra 3.x successfully started, execute the following commands.

1. Verify that the Cassandra version is 3.x:  `nodetool version`
2. Verify that the Cassandra 3.x node is running: `nodetool status`


# Create a keyspace and table, and insert data
1. start CQL shell `cqlsh`

2. create the keyspace:
```cqlsh
CREATE KEYSPACE united_states 
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3};

USE united_states;
```

3. Create the table:

```cqlsh
CREATE TABLE cities_by_state (
    state text,
    name text,
    population int,
    PRIMARY KEY ((state), name)
);
```

4. Insert the top 10 largest U.S. cities by population:
```cqlsh
INSERT INTO cities_by_state (state, name, population) 
  VALUES ('New York','New York City',8622357);
INSERT INTO cities_by_state (state, name, population) 
  VALUES ('California','Los Angeles',4085014);
INSERT INTO cities_by_state (state, name, population) 
  VALUES ('Illinois','Chicago',2670406);
INSERT INTO cities_by_state (state, name, population) 
  VALUES ('Texas','Houston',2378146);
INSERT INTO cities_by_state (state, name, population) 
  VALUES ('Arizona','Phoenix',1743469);
INSERT INTO cities_by_state (state, name, population) 
  VALUES ('Pennsylvania','Philadelphia',1590402);
INSERT INTO cities_by_state (state, name, population) 
  VALUES ('Texas','San Antonio',1579504);
INSERT INTO cities_by_state (state, name, population) 
  VALUES ('California','San Diego',1469490);
INSERT INTO cities_by_state (state, name, population) 
  VALUES ('Texas','Dallas',1400337);
INSERT INTO cities_by_state (state, name, population) 
  VALUES ('California','San Jose',1036242);
INSERT INTO cities_by_state (state, name, population) 
  VALUES ('Essex','Chelmsford',1234567);
```

5. Verify that the data has been loaded:

`SELECT * FROM cities_by_state;`

6. Retrieve all the cities in California:

`SELECT * FROM cities_by_state WHERE state = 'California';`

7.  Exit the CQL shell and clear the screen:
```bash
exit
clear
```

# Verify that the node is ready to be upgraded
In this step, we will verify that the Cassandra 3.x cluster is ready to be upgraded. There are nine factors to consider:

1. Current state: All nodes in the cluster need to be in the ‘Up and Normal’ state. Check that there are no nodes in the cluster that are in a state different to Up and Normal. Verify that all nodes have the UN state:
`nodetool status`

2. Disk space Verify that each node has at least 50% diskspace free:
   
`df -h`

3.  Errors Verify that there are no unresolved errors (also, check for warnings) in the log:

`grep -e "WARN" -e "ERROR" ./logs/system.log`

4. Gossip stability Verify that all entries in the gossip information output have the gossip state STATUS:NORMAL: Check if there are any nodes that have a status other than NORMAL:

`nodetool gossipinfo`

5.  Check for any records that have a status other than NORMAL:

`nodetool gossipinfo | grep STATUS | grep -v NORMAL`

6.  Dropped messages Verify that there were no dropped messages in the past 72 hours:

`nodetool tpstats | grep -A 12 Dropped`

7.  Backup disabled

Verify that all automatic backups have been disabled. This includes disabling Medusa and any scripts that call nodetool snapshot until the upgrade is complete.

8. Repair disabled

Verify that repairs have been disabled. This includes disabling automated repairs in Reaper.

9. Monitoring

Upgrading may result in a temporary reduction in performance, as it simulates a series of temporary node failures. Understanding how the upgrade impacts the performance of the system, both during and after, is crucial when working through the process.

10. Availability

Confirm that areas of the application that require strong consistency are using the LOCAL_QUORUM consistency level and the replication factor of 3.

When LOCAL_QUORUM is used with a replication factor below 3, all replicas must be available for requests to succeed. A rolling restart using this configuration will result in full or partial unavailability while a node is DOWN.

You are now ready to begin the upgrade.


# Prepare the node for migration
In this step, we will prepare the Cassandra 3.x cluster for the upgrade.

1. Take a snapshot of the node in case you need to roll back the upgrade (nodetool snapshot also flushes the memtables to disk):

`nodetool snapshot`  > create snapshot
`nodetool listsnapshots` > list all snapshot

2. Stop the node by finding the PID and calling kill:

`pgrep -u root -f cassandra | xargs kill -9`


2. Use nodetool to verify that the node has been shut down:

`nodetool status`

The node has been shutdown. Continue to the next step.


# Install Cassandra 4.x

1. Download and install Cassandra 4.x:

```bash
wget https://archive.apache.org/dist/cassandra/4.1.6/apache-cassandra-4.1.6-bin.tar.gz

tar -xzf apache-cassandra-4.1.6-bin.tar.gz

rm apache-cassandra-4.1.6-bin.tar.gz

mv apache-cassandra-4.1.6 cassandra4
```

2. Update the PATH variable:

```bash
export CASSANDRA_HOME=/opt/cassandra4
export PATH=/opt/cassandra4/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

# Start Cassandra 4.x
In this step, you will configure cassandra.yaml and start Cassandra 4.x.

1. Change the number of virtual nodes to 256 since the 3.x node had 256 and the 4.x node is set to 16 by default. Set num_tokens to 256 in Cassandra 4.x:

`sed -i 's/num_tokens: 16/num_tokens: 256/' /opt/cassandra4/conf/cassandra.yaml`


2. Point the Cassandra 4.x node to the Cassandra 3.x node data files:

```bash
sed -i 's|# data_file_directories:|data_file_directories:|' /opt/cassandra4/conf/cassandra.yaml
sed -i "s|#     - /var/lib/cassandra/data|    - /opt/cassandra/data/data|" /opt/cassandra4/conf/cassandra.yaml
```

3. Start the Cassandra 4.x node:

`cassandra`
Look for the state jump to NORMAL message to indicate that the node is running.

4. Clear the screen and continue:

`clear`


# change python3 version to 3.11.x as cassandra v4 max support to 3.11 NOT 3.12

```bash
apt install software-properties-common
add-apt-repository ppa:deadsnakes/ppa
apt update
apt install python3.11 python3.11-venv python3.11-dev
update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 1

```


# Verify that the node has been successfully migrated
In this step, you will verify that the Cassandra node has been upgraded and that the data is still available.

1. Verify that the Cassandra version is 4.x: `nodetool version`
2. Verify that the node is in the UP and NORMAL (UN) state: `nodetool status`
3. Verify that there are no errors: `grep -e "WARN" -e "ERROR" cassandra4/logs/system.log`
4. Start the CQL shell: `cqlsh`
5. Use the keyspace: `USE united_states;`
6. Verify that the data is accessible: `SELECT * FROM cities_by_state;`

If you can see the data, you have successfullly upgraded from Cassandra 3.x to 4.x!


```bash
root@cb86c9a97640:/opt/cassandra4/conf# nodetool version
ReleaseVersion: 4.1.6
root@cb86c9a97640:/opt/cassandra4/conf# cqlsh
Connected to Test Cluster at 127.0.0.1:9042
[cqlsh 6.1.0 | Cassandra 4.1.6 | CQL spec 3.4.6 | Native protocol v5]
Use HELP for help.
cqlsh> use united_states ;
cqlsh:united_states> SELECT * FROM cities_by_state;

 state        | name          | population
--------------+---------------+------------
        Essex |    Chelmsford |    1234567
 Pennsylvania |  Philadelphia |    1590402
     New York | New York City |    8622357
      Arizona |       Phoenix |    1743469
     Illinois |       Chicago |    2670406
   California |   Los Angeles |    4085014
   California |     San Diego |    1469490
   California |      San Jose |    1036242
        Texas |        Dallas |    1400337
        Texas |       Houston |    2378146
        Texas |   San Antonio |    1579504
```


# the whole process is point the version v4 `data_file_directories` to v3 `cassandra/data/data`

after added new record in v4, you can see it from v3:
`INSERT INTO cities_by_state (state, name, population) VALUES ('Kent','Kent',2345671);`

```bash
root@7f9f3fc224ac:/opt/cassandra# nodetool version
ReleaseVersion: 3.0.30
root@7f9f3fc224ac:/opt/cassandra# cqlsh
Connected to Test Cluster at 127.0.0.1:9042.
[cqlsh 5.0.1 | Cassandra 3.0.30 | CQL spec 3.4.0 | Native protocol v4]
Use HELP for help.
cqlsh> use united_states ;
cqlsh:united_states> SELECT * FROM cities_by_state;

 state        | name          | population
--------------+---------------+------------
        Essex |    Chelmsford |    1234567
 Pennsylvania |  Philadelphia |    1590402
     New York | New York City |    8622357
         Kent |          Kent |    2345671
      Arizona |       Phoenix |    1743469
     Illinois |       Chicago |    2670406
   California |   Los Angeles |    4085014
   California |     San Diego |    1469490
   California |      San Jose |    1036242
        Texas |        Dallas |    1400337
        Texas |       Houston |    2378146
        Texas |   San Antonio |    1579504
```