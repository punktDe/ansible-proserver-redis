# Replication with multiple nodes

There are two different kinds of clustering concepts possible with Redis, this one only covers [replication](https://redis.io/topics/replication) (One leader, multiple followers). 
Redis [Cluster](https://redis.io/topics/cluster-spec) (multiple leaders, multiple followers) is not possible with this role yet.

In order to set up replication, without the danger of a split brain scenario, at least three different boxes are necessary, each one running an instance of Redis server and an instance of Redis Sentinel.

This role can only use Redis Sentinel on Linux (Ubuntu), because it is not yet possible to run Redis Sentinel as a service with rc-scripts on FreeBSD. 

## Enabling replication

For enabling replication on a node, the variable `redis.replication.enabled` has to be set to a "true" boolean state; e.g.:

```yaml
redis:
  replication:
    enabled: yes
```

### Configuration

There are a few configuration options that have to be set on all the nodes in a replication setup in order to enable clients' access:    

```yaml
redis:
  redis.conf:
    bind: "0.0.0.0" # listens on all interfaces, 
                    # restrict to host's address on a certain 
                    # interface if needed
    port: 6379 # This is redis's standard port
               # adapt where applicable
```

### Leader

In order to mark a node as replication leader it has to be added to the group `redis_replication_leader` in your inventory:

```ini
[redis_replication_leader]
node1
```

Due to compatibility reasons they are still called "master" in Redis terms.

#### Password (Optional)

The leader can require a password towards followers. For this the variable `redis['redis.conf'].requirepass` has to be set to the desired password; e.g.:

```yaml
redis:
  redis.conf:
    requirepass: "secret"
```

### Follower

Replication followers are replicas of the leader's data. 
To mark a nodes as replication followers they have to be added to the group 'redis_replication_follower' in your inventory:

```ini
[redis_replication_follower]
node2
node3
```

Due to compatibility reasons they still are called "slave" or "replica" in Redis terms.

#### Configuration

A follower has to have its leader configured

```yaml
redis:
  redis.conf:
    replicaof: "1.2.3.4 6379" # adapt leader's ip-address & port
```

#### Password (Optional)

If a password is set for the leader to expect, it has to be set for the follower

```yaml
redis:
  redis.conf:
    masterauth: "secret"
```

## Setting up Redis Sentinel

With Redis [Sentinel](https://redis.io/topics/sentinel) it becomes possible to monitor the nodes of a replication setup and let the sentinels establish a quorum between them to vote for a follower to become the new leader, if the leader is disconnected to the replication setup.
This happens automatically, the only thing a Sentinel has to know is the leader during setup. 
In the smallest possible configuration of three machines we let one instance of Redis Sentinel run on each machine next to the respective leader or follower node.

### Enabling Sentinel

In order to enable sentinel the variable `redis.replication.sentinel` has to be set to a "true" boolean state.

```yaml
redis:
  replication:
    sentinel: yes
```

### Configuring Sentinel

For a Sentinel to monitor a replication leader and its followers as well as communicating with other sentinels, some configuration options have to set.
A sentinel will connect with the leader and publish/subscribe to a special key, on which other sentinels are subscribed as well. 
Afterwards the sentinels will communicate in a peer-to-peer protocol.
To configure leader's replica followers is optional, because of autodiscovery features. 

```yaml
redis:
 sentinel.conf:
    bind: 0.0.0.0 # listens on all interfaces, 
                  # restrict to host's address on a certain 
                  # interface if needed
    port: 26379 # This is redis sentinel's standard port
                # adapt where applicable
    protected-mode: "yes"
    "sentinel monitor leader 1.2.3.4 6379": 2 # "leader" is the group's name,
                                              # that has to be used
                                              # for settings regarding this group
                                              # adapt to leader's ip-address & port 
                                              # and the respective quorum
    "sentinel auth-pass leader": secret # Set this if a password has been defined
    "sentinel down-after-milliseconds": "leader 60000"
    "sentinel failover-timeout": "leader 180000"
    "sentinel parallel-syncs": "leader 1"
    "sentinel config-epoch": "leader 0"
    "sentinel leader-epoch": "leader 0"
    "sentinel known-replica leader 2.3.4.5": 6379 # optional, adapt to the replica's ip-address & port
    "sentinel known-replica leader 3.4.5.6": 6379 # optional, adapt to the replica's ip-address & port
```
