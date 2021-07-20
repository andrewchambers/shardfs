# Shardfs

Shardfs is a 9p2000.L proxy server that shards file access based on the top level directory. The sharding
is based on [rendezvous hashing](https://en.wikipedia.org/wiki/Rendezvous_hashing) to minimize rebalancing cost
and is weighted to targets by relative sizes.

## Limitations

- The top level directory does not support renaming files.
- Hard links are not supported.
- Directories are distributed by path hash and do not understand free space.
- The code does not exist yet.

# Basic usage

First setup a [diod](https://github.com/chaos/diod) export on each server you are sharding to.

Create /etc/shardfs-mounts to specify our mounts.
```
# Lines have the format:
# node-id weight transport address export status

1 20GiB tcp 10.100.0.2:1234 shard1 live
2 40GiB tcp 10.100.0.3:1234 shard2 live
3 20GiB tcp 10.100.0.4:1234 shard3 dead
```

Start the server:
```
$ shardfs serve 127.0.0.1:8888
...
```

Mount the filesystem:

```
sudo mount -t 9p -n 127.0.0.1 /shardfs \
    -oport=8888,version=9p2000.L,uname=root,access=user
```

# Rebalancing a cluster

You must perform a rebalance when adding nodes, removing nodes, or changing node relative sizes:

First make your changes to the shardfs-mounts, including marking nodes you no longer want as 'dead'.
Next to copy files to their new nodes, run the rebalance command...

```
$ umount /shardfs
$ shardfs rebalance
```

Once the files are rebalanced, you may remove any dead nodes from the config. To avoid data loss, you must finish the 
rebalance operation before remounting the filesystem.
