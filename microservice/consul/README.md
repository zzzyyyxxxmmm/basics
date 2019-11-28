### Consul -h
```
Usage: consul [--version] [--help] <command> [<args>]

Available commands are:
    acl            Interact with Consul's ACLs
    agent          Runs a Consul agent
    catalog        Interact with the catalog
    config         Interact with Consul's Centralized Configurations
    connect        Interact with Consul Connect
    debug          Records a debugging archive for operators
    event          Fire a new event
    exec           Executes a command on Consul nodes
    force-leave    Forces a member of the cluster to enter the "left" state
    info           Provides debugging information for operators.
    intention      Interact with Connect service intentions
    join           Tell Consul agent to join cluster
    keygen         Generates a new encryption key
    keyring        Manages gossip layer encryption keys
    kv             Interact with the key-value store
    leave          Gracefully leaves the Consul cluster and shuts down
    lock           Execute a command holding a lock
    login          Login to Consul using an auth method
    logout         Destroy a Consul token created with login
    maint          Controls node or service maintenance mode
    members        Lists the members of a Consul cluster
    monitor        Stream logs from a Consul agent
    operator       Provides cluster-level tools for Consul operators
    reload         Triggers the agent to reload configuration files
    rtt            Estimates network round trip time between nodes
    services       Interact with services
    snapshot       Saves, restores and inspects snapshots of Consul server state
    tls            Builtin helpers for creating CAs and certificates
    validate       Validate config files/directories
    version        Prints the Consul version
    watch          Watch for changes in Consul
```

### Consul agent -h
```
Usage: consul agent [options]

  Starts the Consul agent and runs until an interrupt is received. The
  agent represents a single node in a cluster.

HTTP API Options

  -datacenter=<value>
     Datacenter of the agent.

Command Options

  -advertise=<value>
     Sets the advertise address to use.

  -advertise-wan=<value>
     Sets address to advertise on WAN instead of -advertise address.

  -allow-write-http-from=<value>
     Only allow write endpoint calls from given network. CIDR format,
     can be specified multiple times.

  -alt-domain=<value>
     Alternate domain to use for DNS interface.

  -bind=<value>
     Sets the bind address for cluster communication.

  -bootstrap
     Sets server to bootstrap mode.

  -bootstrap-expect=<value>
     Sets server to expect bootstrap mode.

  -check_output_max_size=<value>
     Sets the maximum output size for checks on this agent

  -client=<value>
     Sets the address to bind for client access. This includes RPC, DNS,
     HTTP, HTTPS and gRPC (if configured).

  -config-dir=<value>
     Path to a directory to read configuration files from. This
     will read every file ending in '.json' as configuration in this
     directory in alphabetical order. Can be specified multiple times.

  -config-file=<value>
     Path to a file in JSON or HCL format with a matching file
     extension. Can be specified multiple times.

  -config-format=<value>
     Config files are in this format irrespective of their extension.
     Must be 'hcl' or 'json'

  -data-dir=<value>
     Path to a data directory to store agent state.

  -dev
     Starts the agent in development mode.

  -disable-host-node-id
     Setting this to true will prevent Consul from using information
     from the host to generate a node ID, and will cause Consul to
     generate a random node ID instead.

  -disable-keyring-file
     Disables the backing up of the keyring to a file.

  -dns-port=<value>
     DNS port to use.

  -domain=<value>
     Domain to use for DNS interface.

  -enable-local-script-checks
     Enables health check scripts from configuration file.

  -enable-script-checks
     Enables health check scripts.

  -encrypt=<value>
     Provides the gossip encryption key.

  -grpc-port=<value>
     Sets the gRPC API port to listen on (currently needed for Envoy xDS
     only).

  -hcl=<value>
     hcl config fragment. Can be specified multiple times.

  -http-port=<value>
     Sets the HTTP API port to listen on.

  -join=<value>
     Address of an agent to join at start time. Can be specified
     multiple times.

  -join-wan=<value>
     Address of an agent to join -wan at start time. Can be specified
     multiple times.

  -log-file=<value>
     Path to the file the logs get written to

  -log-level=<value>
     Log level of the agent.

  -log-rotate-bytes=<value>
     Maximum number of bytes that should be written to a log file

  -log-rotate-duration=<value>
     Time after which log rotation needs to be performed

  -log-rotate-max-files=<value>
     Maximum number of log file archives to keep

  -node=<value>
     Name of this node. Must be unique in the cluster.

  -node-id=<value>
     A unique ID for this node across space and time. Defaults to a
     randomly-generated ID that persists in the data-dir.

  -node-meta=<key:value>
     An arbitrary metadata key/value pair for this node, of the format
     `key:value`. Can be specified multiple times.

  -non-voting-server
     (Enterprise-only) This flag is used to make the server not
     participate in the Raft quorum, and have it only receive the data
     replication stream. This can be used to add read scalability to
     a cluster in cases where a high volume of reads to servers are
     needed.

  -pid-file=<value>
     Path to file to store agent PID.

  -protocol=<value>
     Sets the protocol version. Defaults to latest.

  -raft-protocol=<value>
     Sets the Raft protocol version. Defaults to latest.

  -recursor=<value>
     Address of an upstream DNS server. Can be specified multiple times.

  -rejoin
     Ignores a previous leave and attempts to rejoin the cluster.

  -retry-interval=<value>
     Time to wait between join attempts.

  -retry-interval-wan=<value>
     Time to wait between join -wan attempts.

  -retry-join=<value>
     Address of an agent to join at start time with retries enabled. Can
     be specified multiple times.

  -retry-join-wan=<value>
     Address of an agent to join -wan at start time with retries
     enabled. Can be specified multiple times.

  -retry-max=<value>
     Maximum number of join attempts. Defaults to 0, which will retry
     indefinitely.

  -retry-max-wan=<value>
     Maximum number of join -wan attempts. Defaults to 0, which will
     retry indefinitely.

  -segment=<value>
     (Enterprise-only) Sets the network segment to join.

  -serf-lan-bind=<value>
     Address to bind Serf LAN listeners to.

  -serf-lan-port=<value>
     Sets the Serf LAN port to listen on.

  -serf-wan-bind=<value>
     Address to bind Serf WAN listeners to.

  -serf-wan-port=<value>
     Sets the Serf WAN port to listen on.

  -server
     Switches agent to server mode.

  -server-port=<value>
     Sets the server port to listen on.

  -syslog
     Enables logging to syslog.

  -ui
     Enables the built-in static web UI server.

  -ui-content-path=<value>
     Sets the external UI path to a string. Defaults to: /ui/

  -ui-dir=<value>
     Path to directory containing the web UI resources.
```