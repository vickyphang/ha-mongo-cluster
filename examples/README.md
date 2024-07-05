## HA MongoDB Sharded Cluster (Unstable)
> Temporary, not enough resource to continue the research. The procedure below still a theory.

> There are stil many typos, like config <-> configsrv

<p align="center"> <img src="/images/topology.png"> </p>

### 1. Deploy all resource
- Deploy mongos router
    ```bash
    kubectl apply -f mongos-router.yaml
    ```
- Deploy config server replica set
    ```bash
    kubectl apply -f configsrv-rs.yaml
    ```
- Deploy shard replica sets
    ```bash
    kubectl apply -f shard1-rs.yaml
    kubectl apply -f shard2-rs.yaml
    ```

### 2. Manually configure sharding
- Configure config server replica set
    ```bash
    # connect to the mongodb
    mongo --host configsrv-replicaset --port 27017 -u admin -p 'your-password-here' --authenticationDatabase admin

    # Initiate the config server replica set
    rs.initiate({
        _id: "configReplSet",
        configsvr: true,
        members: [
            { _id: 0, host: "config-replicaset-0.config-replicaset.default.svc.cluster.local:27017" },
            { _id: 1, host: "config-replicaset-1.config-replicaset.default.svc.cluster.local:27017" },
            { _id: 2, host: "config-replicaset-2.config-replicaset.default.svc.cluster.local:27017" }
        ]
    });
    ```
- Configure shard1 and shard2 replica sets
    ```bash
    # connect to both mongodb
    mongo --host shard1-replicaset --port 27017 -u admin -p 'your-password-here' --authenticationDatabase admin

    # Initiate both shard replica sets
    rs.initiate({
        _id: "shard1ReplSet",
        members: [
            { _id: 0, host: "shard1-replicaset-0.shard1-replicaset.default.svc.cluster.local:27017" },
            { _id: 1, host: "shard1-replicaset-1.shard1-replicaset.default.svc.cluster.local:27017" },
            { _id: 2, host: "shard1-replicaset-2.shard1-replicaset.default.svc.cluster.local:27017" }
        ]
    });
    ```
- Configure mongos router
    ```bash
    # Add shards to the cluster
    sh.addShard("shard1ReplSet/shard1-replicaset-0.shard1-replicaset.default.svc.cluster.local:27017")
    sh.addShard("shard2ReplSet/shard2-replicaset-0.shard2-replicaset.default.svc.cluster.local:27017")

    # Enable sharding for databases and collections
    sh.enableSharding("myDatabase")
    sh.shardCollection("myDatabase.myCollection", { shardKey: 1 })
    ```