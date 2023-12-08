## What nodes to install on

### `pbm-agent`

Install [`pbm-agent`](../details/pbm-agent.md) on all servers that have `mongod` nodes in the
MongoDB cluster (or non-sharded replica set). You don't need to start it on the `arbiter` node, since it doesn’t have the data set.

### `pbm` CLI

You can install [`pbm` CLI](../details/cli.md) on any or all servers or desktop computers you wish to use it from. Those computers must not be network-blocked from accessing the MongoDB cluster.