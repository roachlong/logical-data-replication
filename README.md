# logical-data-replication
This is a quick setup to demonstrate replication in CRDB across a set of tables between two clusters.  **Note** that LDR is available in v24.3 onwards.

It is assumed that you have a primary and secondary cluster with enterprise licenses running and that you have ssh access to both.  Cockroach SQL commands are executed locally against your cluster, but there are also steps to copy source (primary) certs to the target (secondary) that will execute commands remotely on one of the nodes in your cluster with ```ssh -t user@node ...```.

## 1 Configure Clusters For Replication
Execute the following commands to enable replication on your clusters.  This is a one time setup to store to confirm the configruation settings on the database.  You'll need the following information for each cluster:
1) **Database Name**: the name of the database in your connection string
2) **Database User**: the username that will be used to connect to the database and to configure replication, fyi dont' use root
3) **Primary Node**: the hostname or IP address for the primary node on your cluster
4) **Connection String**: the database url used to connect to the cluster
5) **System Account**: the username that will be used to ssh into the server without a password, it is assumed you have a public ssh keys installed and enabled
```
./replicate.sh config_primary
./replicate.sh config_secondary
```

## 2 Execute TPCC Workload
Next we can execute a sample workload on your primary database.  I would run this in a separate terminal.  It can be started before, duirng or after replication starts and you can run the workload multiple times.
```
./replicate.sh exec_workload
```

## 3 Test Logical Data Replication
0) First we just need to establish a connection betwen the target (secondary cluster) and to source (primary cluster).  Once this is done the next two steps can be repeated as often as needed.
1) Next we can start replication between the specific tables for our workload and immediated monitor the flow of data between the two systems.  Note that this configuration is only setup for one-way replication between the source and target tables.
2) At any point we can stop replication, which would normally pause teh flow of data, but here we're cleaning up the target system so that we can restart the flow of data from scratch with step 1 above.  We DO NOT need to execute step 0 a second time to establish a connection between the clusters.
```
# 0) this first step is a one-time setup to create the connection from target to source
./replicate.sh setup_ldr

# 1) this next step will start replication between the requested database objects
./replicate.sh start_ldr

# 2) and this step will stop replication and remove the data form the target system
./replicate.sh stop_ldr
```

## Notes
The following steps are for internal scripts to stand-up and tear-down clusters using roachprod inside Cockroach Labs.

Stand-up:
```
(cd ../cockroachdb-starter/simple-bulk-insert; ./01_init_nodes.sh primary)
(cd ../cockroachdb-starter/simple-bulk-insert; ./01_init_nodes.sh secondary)
mkdir -p certs && cp -r ../cockroachdb-starter/simple-bulk-insert/certs .
```

Execute replication steps...

Tear-down:
```
(cd ../cockroachdb-starter/simple-bulk-insert; ./04_shutdown.sh secondary)
(cd ../cockroachdb-starter/simple-bulk-insert; ./04_shutdown.sh primary)
rm -rf src-certs certs settings
```
