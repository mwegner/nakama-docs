# Configuration

A YAML configuration file configures many aspects of how your Nakama server runs. You can run Nakama without specifying a configuration file and rely on the the default settings instead.

## Specifying a config file
You can specify a configuration file at run-time using --config flag.

```shell fct_label="Shell"
nakama --config my-special-config.yml
```

If you are running Nakama via Docker-Compose, you'll need to bind a folder in your machine so that it's available in Docker. Follow this guide to [setup folder binding](install-docker-quickstart.md#data).

## Server ports

Nakama is a very flexible system. You can exchange data with the server over gRPC, HTTP, Websockets and rUDP. Due to this flexibilty, Nakama requires 4 ports to be available to bind to:

- HTTP API server on port 7350. *Port can be changed in the config.*
- HTTP API server powering the embedded developer console on port 7351. *Port can be changed in the config.*
- gRPC API server on port 7349. *Port is chosen based on the API server port.*
- gRPC API server for the embedded console on port 7348. *Port is chosen based on the API server port.*

We'll be reducing the port requirement in the future releases.

## Common properties

There are a few configuration properties that need to be changed in most environments. The full list of configurations is at the [bottom of the page](#server-configuration).

| Parameter | Description
| --------- | -----------
| `name` | Nakama node name (must be unique) - It will default to `nakama`.
| `data_dir` | An absolute path to a writeable folder where Nakama will store its data, including logs. Default value is the working directory that Nakama was started on.
| `runtime.path` | Path of modules to scan and load. Defaults to `data_dir/modules`.
| `database.address` | List of database nodes to connect to. It should follow the form of `username:password@address:port/dbname` (`postgres://` protocol is appended to the path automatically). Defaults to `root@localhost:26257`.
| `socket.server_key` | Server key to use to establish a connection to the server. Default value is `defaultkey`.
| `runtime.http_key` | Key is used to protect the server's runtime HTTP invocations. Default value is `defaultkey`.
| `session.encryption_key` | The encryption key used to produce the client token. Default value is `defaultencryptionkey`.
| `session.token_expiry_sec` | Session token expiry in seconds. Default value is 60.


!!! warning "Production settings"
    You must change the values of **`socket.server_key`**, **`session.encryption_key`** and **`runtime.http_key`** before you deploy Nakama to a live production environment.

## Server Configuration

Nakama has various configuration options to make it as flexible as possible for various use cases and deployment environments.

Nakama ships with sane default values for all config options, therefore you'll only need to override a subset of the options. You can also setup your own config file, and override the values in the config file via command-line flags. For instance, to override Runtime Path:

```shell fct_label="Shell"
nakama --runtime.path /tmp/my-modules
```

If fields are not specific, default values will be used. For more information on how to override flags, have a look at the [server command-line](install-server-cli.md) page.

!!! tip "Override configuration"
    Every configuration option can set from a config file, as a command line flag or both where the command-line argument takes precedence and will override the configuration values.

| Parameter | Flag | Description
| --------- | ---- | -----------
| `name` | `name` | Nakama node name (must be unique) - It will default to `nakama`. This name is also used in the log files.
| `data_dir` | `data_dir` | An absolute path to a writeable folder where Nakama will store its data, including logs. Default value is the working directory that Nakama was started on.
| `shutdown_grace_sec` | `shutdown_grace_sec` | Maximum number of seconds to wait for the server to complete work before shutting down. If 0 the server will shut down immediately when it receives a termination signal. Default value is `0`.

### Cluster

This section configures how the nodes should connect to each to other form a cluster.

!!! tip "Nakama Enterprise Only"
    The following configuration options are available only in the Nakama Enterprise version of the Nakama server

    Nakama is designed to run in production as a highly available cluster. You can start a cluster locally on your development machine if you’re running [Nakama Enterprise](https://heroiclabs.com/nakama-enterprise). In production you can use either Nakama Enterprise or our [Managed Cloud](https://heroiclabs.com/managed-cloud) service.

| Parameter | Flag | Description
| --------- | ---- | -----------
| `gossip_bindaddr` | `cluster.gossip_bindaddr` | Interface address to bind Nakama to for discovery. By default listening on all interfaces.
| `gossip_bindport` | `cluster.gossip_bindport` | Port number to bind Nakama to for discovery. Default value is 7352.
| `join` | `cluster.join` | List of hostname and port of other Nakama nodes to connect to.
| `max_message_size_bytes` | `cluster.max_message_size_bytes` | Maximum amount of data in bytes allowed to be sent between Nakama nodes per message. Default value is 4194304.
| `rpc_port` | `cluster.rpc_port` | Port number to use to send data between Nakama nodes. Default value is 7353.

### Console

This section defined the configuration related for the embedded developer console.

| Parameter | Flag | Description
| --------- | ---- | -----------
| `password` | `console.password` | Password for the embedded console. Default is "password".
| `port` | `console.port` | The port for accepting connections for the embedded console, listening on all interfaces. Default value is 7351.
| `username` | `console.username` | Username for the embedded console. Default is "admin".

### Database

Nakama requires a CockroachDB server instance to be available. Nakama creates and manages its own database called `nakama` within the CockroachDB database.

| Parameter | Flag | Description
| --------- | ---- | -----------
| `address` | `database.address` | List of database nodes to connect to. It should follow the form of `username:password@address:port/dbname` (`postgres://` protocol is appended to the path automatically). Defaults to `root@localhost:26257`.
| `conn_max_lifetime_ms` | `database.conn_max_lifetime_ms` | Time in milliseconds to reuse a database connection before the connection is killed and a new one is created.. Default value is 0 (indefinite).
| `max_idle_conns` | `database.max_idle_conns` | Maximum number of allowed open but unused connections to the database. Default value is 100.
| `max_open_conns` | `database.max_open_conns` | Maximum number of allowed open connections to the database. Default value is 0 (no limit).

!!! tip "Database addresses"
    You can pass in multiple database addresses to Nakama via command like:

    ```
    nakama --database.address "root@db1:26257" --database.address "root@db2:26257"
    ```

### Leaderboard

You can change configuration options related to the leaderboard and tournament system.

| Parameter | Flag | Description
| --------- | ---- | -----------
| `blacklist_rank_cache` | `leaderboard.blacklist_rank_cache` | Disable rank cache for leaderboards with matching leaderboard names.

!!! tip "Disable rank cache"
    To disable rank cache entirely, use `*`, otherwise leave blank to enable rank cache.

### Logger

Nakama produces logs in JSON format so various systems can interact with the logs. By default they are written to the standard out (console).

| Parameter | Flag | Description
| --------- | ---- | -----------
| `compress` | `logger.compress` | This determines if the rotated log files should be compressed using gzip.
| `file` | `logger.file` | Log output to a file (as well as `stdout` if set). Make sure that the directory and the file is writable.
| `level` | `logger.level` | Minimum log level to produce. Values are `debug`, `info`, `warn` and `error`. Default is `info`.
| `local_time` | `logger.local_time` | This determines if the time used for formatting the timestamps in backup files is the computer's local time. The default is to use UTC time.
| `max_age` | `logger.max_age` | The maximum number of days to retain old log files based on the timestamp encoded in their filename. The default is not to remove old log files based on age.
| `max_backups` | `logger.max_backups` | The maximum number of old log files to retain. The default is to retain all old log files (though `max_age` may still cause them to get deleted.)
| `max_size` | `logger.max_size` | The maximum size in megabytes of the log file before it gets rotated. It defaults to 100 megabytes. Default is 100.
| `rotation` | `logger.rotation` | Rotate log files. Default is false.
| `stdout` | `logger.stdout` | Redirect logs to console standard output. The log file will no longer be used. Default is `true`.

The standard startup log messages will always be printed to the console irrespective of the value of `logger.stdout` field.

### Match

You can change configuration options related to the authoritative multiplayer runtime.

| Parameter | Flag | Description
| --------- | ---- | -----------
| `call_queue_size` | `match.call_queue_size` | Size of the authoritative match buffer that sequences calls to match handler callbacks to ensure no overlaps. Default 128.
| `deferred_queue_size` | `match.deferred_queue_size` | Size of the authoritative match buffer that holds deferred message broadcasts until the end of each loop execution. Default 128.
| `input_queue_size` | `match.input_queue_size` | Size of the authoritative match buffer that stores client messages until they can be processed by the next tick. Default 128.
| `join_attempt_queue_size` | `match.join_attempt_queue_size` | Size of the authoritative match buffer that limits the number of in-progress join attempts. Default 128.
| `join_marker_deadline_ms` | `match.join_marker_deadline_ms` | Deadline in milliseconds that client authoritative match joins will wait for match handlers to acknowledge joins. Default 5000.

### Metrics

Nakama produces metrics information. This information can be exported to Stackdriver or Prometheus.

| Parameter | Flag | Description
| --------- | ---- | -----------
| `namespace` | `metrics.namespace` | Namespace for Prometheus or prefix for Stackdriver metrics. It will always prepend node name. Default value is empty.
| `prometheus_port` | `metrics.prometheus_port` | Port to expose Prometheus. Default value is '0' which disables Prometheus exports.
| `reporting_freq_sec` | `metrics.reporting_freq_sec` | Frequency of metrics exports. Default is 60 seconds.
| `stackdriver_projectid` | `metrics.stackdriver_projectid` | This is the identifier of the Stackdriver project the server is uploading the stats data to. Setting this enables metrics to be exported to Stackdriver.

Ensure that metrics exports are protected as they contain sensitive server information.

### Runtime

Options related to Lua-based runtime engine.

| Parameter | Flag | Description
| --------- | ---- | -----------
| `call_stack_size` | `runtime.call_stack_size` | Size of each runtime instance's call stack. Default 128.
| `env` | `runtime.env` | List of Key-Value properties that are exposed to the Runtime scripts as environment variables.
| `http_key` | `runtime.http_key` | A key used to authenticate HTTP Runtime invocations. Default value is `defaultkey`.
| `max_count` | `runtime.max_count` | Maximum number of runtime instances to allocate. Default 256.
| `min_count` | `runtime.min_count` | Minimum number of runtime instances to allocate. Default 16.
| `path` | `runtime.path` | Path of modules for the server to scan and load at startup. Default value is `data_dir/modules`.
| `registry_size` | `runtime.registry_size` | Size of each runtime instance's registry. Default 512.

!!! warning "Important"
    You must change `http_key` before going live with your app!

!!! tip "Runtime env value"
    The runtime environment is a key-value pair. They are separated by the `=` character like this:

    ```
    nakama --runtime.env "key=value" --runtime.env "key2=value2" --runtime.env "key3=valuecanhave=sign"
    ```

### Session

You can change configuration options related to each user session, such as the encryption key used to create the token.

| Parameter | Flag | Description
| --------- | ---- | -----------
| `encryption_key` | `session.encryption_key` | The encryption key used to produce the client token. Default value is `defaultencryptionkey`.
| `token_expiry_sec` | `session.token_expiry_sec` | Token expiry in seconds. Default value is 60.

!!! warning "Important"
    You must change `encryption_key` before going live with your app!

### Social

Nakama can connect to various social networks to fetch user information. It can also act as a notification center for delivering and persisting notifications.

#### Steam
Configure Steam network settings. Facebook, Google and GameCenter don't require any server settings.

| Parameter | Flag | Description
| --------- | ---- | -----------
| `app_id` | `steam.app_id` | Steam App ID.
| `publisher_key` | `steam.publisher_key` | Steam Publisher Key.

### Socket

Options related to connection socket and transport protocol between the server and clients.

| Parameter | Flag | Description
| --------- | ---- | -----------
| `address` | `socket.address` | The IP address of the interface to listen for client traffic on. Default listen on all available addresses/interfaces.
| `idle_timeout_ms` | `socket.idle_timeout_ms` | Maximum amount of time in milliseconds to wait for the next request when keep-alives are enabled. Used for HTTP connections. Default value is 60000.
| `max_message_size_bytes` | `socket.max_message_size_bytes` | Maximum amount of data in bytes allowed to be read from the client socket per message. Used for real-time, gRPC and HTTP connections. Default value is 4096.
| `outgoing_queue_size` | `socket.outgoing_queue_size` | The maximum number of messages waiting to be sent to the client. If this is exceeded the client is considered too slow and will disconnect. Used when processing real-time connections. Default value is 16.
| `ping_backoff_threshold` | `socket.ping_backoff_threshold` | Minimum number of messages received from the client during a single ping period that will delay the sending of a ping until the next ping period, to avoid sending unnecessary pings on regularly active connections. Default value is 20.
| `ping_period_ms` | `socket.ping_period_ms` | Time in milliseconds to wait between client ping messages. This value must be less than the `pong_wait_ms`. Used for real-time
connections. Default value is 8000.
| `pong_wait_ms`  | `socket.pong_wait_ms` | Time in milliseconds to wait for a pong message from the client after sending a ping. Used for real-time connections. Default value is 10000.
| `port` | `socket.port` | The port for accepting connections from the client, listening on all interfaces. Default value is 7350.
| `protocol` | `socket.protocol` | The network protocol to listen for traffic on. Possible values are `tcp` for both IPv4 and IPv6, `tcp4` for IPv4 only, or `tcp6` for IPv6 only. Default `tcp`."
| `read_timeout_ms` | `socket.read_timeout_ms` | Maximum duration in milliseconds for reading the entire request. Used for HTTP connections. Default value is 10000.
| `server_key` | `socket.server_key` | Server key to use to establish a connection to the server. Default value is `defaultkey`.
| `ssl_certificate` | `socket.ssl_certificate` | Path to certificate file if you want the server to use SSL directly. Must also supply ssl_private_key. NOT recommended for production use.
| `ssl_private_key` | `socket.ssl_private_key` | Path to private key file if you want the server to use SSL directly. Must also supply ssl_certificate. NOT recommended for production use.
| `write_timeout_ms` | `socket.write_timeout_ms` | Maximum duration in milliseconds before timing out writes of the response. Used for HTTP connections. Default value is 10000.
| `write_wait_ms` | `socket.write_wait_ms` | Time in milliseconds to wait for an ack from the client when writing data. Used for real-time connections. Default value is 5000.

!!! warning "Important"
    You must change `server_key` before going live with your app!

<!--
!!! info "Public Address"
    Public Address is the direct addressable IP address of your server. This value is cached in session tokens with clients to enable fast reconnects. If the IP address changes clients will need to re-authenticate with the server.
-->

### Tracker

You can change configuration options related to session tracking.

| Parameter | Flag | Description
| --------- | ---- | -----------
| `event_queue_size` | `tracker.event_queue_size` | Size of the tracker presence event buffer. Increase if the server is expected to generate a large number of presence events in a short time. Default value is 1024.

!!! tip "Nakama Enterprise Only"
    The following configuration options are available only in the Nakama Enterprise version of the Nakama server

    Nakama is designed to run in production as a highly available cluster. You can start a cluster locally on your development machine if you’re running [Nakama Enterprise](https://heroiclabs.com/nakama-enterprise). In production you can use either Nakama Enterprise or our [Managed Cloud](https://heroiclabs.com/managed-cloud) service.

| Parameter | Flag | Description
| --------- | ---- | -----------
| `broadcast_period_ms` | `tracker.broadcast_period_ms` | Time in milliseconds between tracker presence replication broadcasts to each cluster node. Default value is 1500.
| `clock_sample_periods` | `tracker.clock_sample_periods` | Number of broadcasts before a presence transfer will be requested from a cluster node if one is not received as expected. Default value is 2.
| `max_delta_sizes` | `tracker.max_delta_sizes` | Number of deltas and maximum presence count per delta for presence snapshots used to broadcast minimal subsets of presence data to cluster nodes. Default values are 100, 1000 and 10000.
| `max_silent_periods` | `tracker.max_silent_periods` | Maximum number of missed broadcasts before a cluster node's presences are considered down. Default value is 10.
| `permdown_period_ms` | `tracker.permdown_period_ms` | Time in milliseconds since last broadcast before a cluster node's presences are considered permanently down and will be removed. Default value is 1200000.

<!--

### Purchase

Nakama can verify in-app purchases by connecting to various stores and keeps a ledger of valid purchases. This is useful for preventing common in-app purchase replay attacks with valid receipts, as well as restoring purchases for user accounts as needed.

#### Apple

Apple In-App Purchase configuration

| Parameter    | Flag                  | Description
| ---------    | ----                  | -----------
| `password`   | `apple.password`      | Your application's shared secret.
| `production` | `apple.production`    | The order in which the reciept verification server will be contacted. Set `false` for test apps and `true` for live apps. Default is `false`.
| `timeout_ms` | `apple.timeout_ms`    | Connection timeout to connect to Apple services. Default value is 1500.

#### Google

Google In-App Purchase configuration

| Parameter          | Flag                       | Description
| ---------          | ----                       | -----------
| `package`          | `google.package`           | The package name the app is published under, such as `com.myapp.testapp`.
| `service_key_file` | `google.service_key_file`  | Absolute file path to the service key JSON file.
| `timeout_ms`       | `google..timeout_ms`       | Connection timeout to connect to Google services. Default value is 1500.

-->


## Example File

You can use the entire file or just a subset of the configuration.

```yaml
name: nakama-node-1
data_dir: "./data/"

logger:
  stdout: false
  level: "warn"
  file: "/tmp/path/to/logfile.log"

metrics:
  reporting_freq_sec: 60
  namespace: ""
  stackdriver_projectid: ""
  prometheus_port: 0

database:
  address:
    - "root@localhost:26257"
  conn_max_lifetime_ms: 0
  max_open_conns: 0
  max_idle_conns: 100

runtime:
  env:
    - "example_apikey=example_apivalue"
    - "encryptionkey=afefa==e332*u13=971mldq"
  path: "/tmp/modules/folders"
  http_key: "defaultkey"

socket:
  server_key: "defaultkey"
  port: 7350
  max_message_size_bytes: 4096 # bytes
  read_timeout_ms: 10000
  write_timeout_ms: 10000
  idle_timeout_ms: 60000
  write_wait_ms: 5000
  pong_wait_ms: 10000
  ping_period_ms: 8000 # Must be less than pong_wait_ms
  outgoing_queue_size: 16

session:
  encryption_key: "defaultencryptionkey"
  token_expiry_sec: 60

social:
  steam:
    publisher_key: ""
    app_id: 0

console:
  port: 7351
  username: "admin"
  password: "password"

cluster:
  join:
    - "10.0.0.2:7352"
    - "10.0.0.3:7352"
  gossip_bindaddr: "0.0.0.0"
  gossip_bindport: 7352
  rpc_port: 7353
```
