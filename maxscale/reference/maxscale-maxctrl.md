# MaxScale MaxCtrl

## MaxCtrl

MaxCtrl is a command line administrative client for MaxScale which uses
the MaxScale REST API for communication. It has replaced the legacy MaxAdmin
command line client that is no longer supported or included.

By default, the MaxScale REST API listens on port 8989 on the local host. The
default credentials for the REST API are `admin:mariadb`. The users used by the
REST API are the same that are used by the MaxAdmin network interface. This
means that any users created for the MaxAdmin network interface should work with
the MaxScale REST API and MaxCtrl.

For more information about the MaxScale REST API, refer to the
[REST API documentation](maxscale-rest-api/maxscale-rest-api.md).
and the
[Configuration guide](../maxscale-management/deployment/maxscale-configuration-guide.md).

## Limitations

* MaxCtrl does not work when used from a SystemD unit with MemoryDenyWriteExecute=true.

## .maxctrl.cnf

If the file `~/.maxctrl.cnf` exists, maxctrl will use any values in the
section `[maxctrl]` as defaults for command line arguments. For instance,
to avoid having to specify the user and password on the command line,
create the file `.maxctrl.cnf` in your home directory, with the following
content:

```
[maxctrl]
user = my-name
password = my-password
```

Note that all access rights to the file must be removed from everybody else
but the owner. MaxCtrl refuses to use the file unless the rights have been
removed.

Another file from which to read the defaults can be specified with the 
`--config=my-file` / `-c my-file` option.

## Common command line options

The following common command line options are always available:

```
Global Options:
  -c, --config     MaxCtrl configuration file  [string] [default: "~/.maxctrl.cnf"]
  -u, --user       Username to use  [string] [default: "admin"]
  -p, --password   Password for the user. To input the password manually, 
                   use -p '' or --password=''  [string] [default: "mariadb"]
  -h, --hosts      List of MaxScale hosts. The hosts must be in HOST:PORT format 
                   and each value must be separated by a comma.  [string] [default: "127.0.0.1:8989"]
  -t, --timeout    Request timeout in plain milliseconds, e.g '-t 1000', 
                   or as duration with suffix [h|m|s|ms], e.g. '-t 10s'  [string] [default: "10000"]
  -q, --quiet      Silence all output. Ignored while in interactive mode.  [boolean] [default: false]
      --tsv        Print tab separated output  [boolean] [default: false]
      --skip-sync  Disable configuration synchronization for this command  [boolean] [default: false]

HTTPS/TLS Options:
  -s, --secure                  Enable HTTPS requests  [boolean] [default: false]
      --tls-key                 Path to TLS private key  [string]
      --tls-passphrase          Password for the TLS private key  [string]
      --tls-cert                Path to TLS public certificate  [string]
      --tls-ca-cert             Path to TLS CA certificate  [string]
  -n, --tls-verify-server-cert  Whether to verify server TLS certificates  [boolean] [default: true]

General Options:
      --version  Show version number  [boolean]
      --help     Show help  [boolean]
```

## Commands

### list

#### list servers

List all servers in MaxScale.

Usage:

```
maxctrl [options] list servers
```

Result fields:

|  Field       | Description                             |
|  -----       | --------------------------------------- |
|  Server      | Server name                             |
|  Address     | Address where the server listens        |
|  Port        | The port on which the server listens    |
|  Connections | Current connection count                |
|  State       | Server state                            |
|  GTID        | Current value of @@gtid_current_pos     |
|  Monitor     | The monitor for this server             |

Example output:

```
$  maxctrl list servers
┌───────────┬─────────┬──────┬─────────────┬──────────────────────┬─────────────┬─────────┐
│ Server    │ Address │ Port │ Connections │ State                │ GTID        │ Monitor │
├───────────┼─────────┼──────┼─────────────┼──────────────────────┼─────────────┼─────────┤
│ master    │ master  │ 3306 │ 0           │ Maintenance, Running │ 23-11-39    │ Monitor │
├───────────┼─────────┼──────┼─────────────┼──────────────────────┼─────────────┼─────────┤
│ replica-1 │ slave-1 │ 3306 │ 0           │ Master, Running      │ 23-12-14957 │ Monitor │
├───────────┼─────────┼──────┼─────────────┼──────────────────────┼─────────────┼─────────┤
│ replica-2 │ slave-2 │ 3306 │ 0           │ Slave, Running       │ 23-12-14957 │ Monitor │
└───────────┴─────────┴──────┴─────────────┴──────────────────────┴─────────────┴─────────┘
```

#### list services

List all services and the servers they use.

Usage:

```
maxctrl [options] list services
```

Output fields:

|  Field             | Description |
|  -----             | ----------- |
|  Service           | Service name |
|  Router            | Router used by the service |
|  Connections       | Current connection count |
|  Total Connections | Total connection count |
|  Targets           | Targets that the service uses |


Exaple output:

```
$ maxctrl list services
┌────────────────────┬────────────────┬─────────────┬───────────────────┬──────────────────────────────┐
│ Service            │ Router         │ Connections │ Total Connections │ Targets                      │
├────────────────────┼────────────────┼─────────────┼───────────────────┼──────────────────────────────┤
│ Read-Write-Service │ readwritesplit │ 0           │ 0                 │ master, replica-1, replica-2 │
└────────────────────┴────────────────┴─────────────┴───────────────────┴──────────────────────────────┘
```

#### list listeners

List listeners of all services. If a service is given, only listeners for that service are listed.

Usage: 

```
maxctrl [options] list listeners [service]
```

Output fields:

|  Field   | Description |
|  -----   | ----------- |
|  Name    | Listener name |
|  Port    | The port where the listener listens |
|  Host    | The address or socket where the listener listens |
|  State   | Listener state |
|  Service | Service that this listener points to |

Example Output:

```
$ maxctrl list listeners Read-Write-Service
┌─────────────────────┬──────┬──────┬─────────┬────────────────────┐
│ Name                │ Port │ Host │ State   │ Service            │
├─────────────────────┼──────┼──────┼─────────┼────────────────────┤
│ Read-Write-Listener │ 3306 │ ::   │ Running │ Read-Write-Service │
└─────────────────────┴──────┴──────┴─────────┴────────────────────┘
```

#### list monitors

List all monitors in MaxScale.

Usage:

```
maxctrl [options] list monitors
```

Output fields:

|  Field   | Description |
|  -----   | ----------- |
|  Monitor | Monitor name |
|  State   | Monitor state |
|  Servers | The servers that this monitor monitors |

Example output:

```
$ maxctrl list monitors
┌─────────┬─────────┬──────────────────────────────┐
│ Monitor │ State   │ Servers                      │
├─────────┼─────────┼──────────────────────────────┤
│ Monitor │ Running │ master, replica-1, replica-2 │
└─────────┴─────────┴──────────────────────────────┘
```

#### list sessions

List all client sessions.

Usage:

```
maxctrl list [options] sessions
```

Command specific options:
```
  --rdns     Perform a reverse DNS lookup on client IPs  [boolean] [default: false]
```

Output fields:

|  Field     | Description |
|  -----     | ----------- |
|  Id        | Session ID |
|  User      | Username |
|  Host      | Client host address |
|  Connected | Time when the session started |
|  Idle      | How long the session has been idle, in seconds |
|  Service   | The service where the session connected |
|  Memory    | Memory usage (not exhaustive) |

Example output:

```
$ maxctrl list sessions
┌────┬────────┬──────────────┬─────────────────────────┬──────┬────────────────────┬────────┬──────────────┐
│ Id │ User   │ Host         │ Connected               │ Idle │ Service            │ Memory │ I/O-Activity │
├────┼────────┼──────────────┼─────────────────────────┼──────┼────────────────────┼────────┼──────────────┤
│ 3  │ myuser │ 10.23.63.100 │ 11/22/2025, 10:31:13 AM │ 1.6  │ Read-Write-Service │ 133098 │ 14           │
├────┼────────┼──────────────┼─────────────────────────┼──────┼────────────────────┼────────┼──────────────┤
│ 2  │ myuser │ 10.23.63.100 │ 11/22/2025, 10:31:07 AM │ 7.8  │ Read-Write-Service │ 133098 │ 14           │
└────┴────────┴──────────────┴─────────────────────────┴──────┴────────────────────┴────────┴──────────────┘
```

#### list filters

List all filters in MaxScale.

Usage: 

```
maxctrl [options] list filters
```

Output fields:

|  Field   | Description |
|  -----   | ----------- |
|  Filter  | Filter name |
|  Service | Services that use the filter |
|  Module  | The module that the filter uses |

Example output:

```
TODO
```

#### list modules

List all currently loaded modules.

Usage:

```
maxctrl [options] list modules
```

Output fields:

|  Field   | Description |
|  -----   | ----------- |
|  Module  | Module name |
|  Type    | Module type |
|  Version | Module version |
  
Example output:

```
$ maxctrl list modules
┌─────────────────┬───────────────┬─────────┐
│ Module          │ Type          │ Version │
├─────────────────┼───────────────┼─────────┤
│ maxscale        │ maxscale      │ 25.01.4 │
├─────────────────┼───────────────┼─────────┤
│ servers         │ servers       │ 25.01.4 │
├─────────────────┼───────────────┼─────────┤
│ MariaDBAuth     │ Authenticator │ V2.1.0  │
├─────────────────┼───────────────┼─────────┤
│ mariadbmon      │ Monitor       │ V1.5.0  │
├─────────────────┼───────────────┼─────────┤
│ MariaDBProtocol │ Protocol      │ V1.1.0  │
├─────────────────┼───────────────┼─────────┤
│ pp_sqlite       │ Parser        │ V1.0.0  │
├─────────────────┼───────────────┼─────────┤
│ readwritesplit  │ Router        │ V1.1.0  │
└─────────────────┴───────────────┴─────────┘
```

#### list threads

List all worker threads.

Usage:

```
maxctrl [options] list threads
```

Output fields:

|  Field       | Description |
|  -----       | ----------- |
|  Id          | Thread ID |
|  Current FDs | Current number of managed file descriptors |
|  Total FDs   | Total number of managed file descriptors |
|  Load (1s)   | Load percentage over the last second |
|  Load (1m)   | Load percentage over the last minute |
|  Load (1h)   | Load percentage over the last hour |

Example output:

```
$ maxctrl list threads
┌────┬─────────────┬───────────┬───────────┬───────────┬───────────┐
│ Id │ Current FDs │ Total FDs │ Load (1s) │ Load (1m) │ Load (1h) │
├────┼─────────────┼───────────┼───────────┼───────────┼───────────┤
│ 0  │ 10          │ 10        │ 0         │ 0         │ 0         │
├────┼─────────────┼───────────┼───────────┼───────────┼───────────┤
│ 1  │ 4           │ 5         │ 0         │ 0         │ 0         │
└────┴─────────────┴───────────┴───────────┴───────────┴───────────┘
```

#### list users

List the users that can be used to connect to the MaxScale REST API.

Usage:

```
maxctrl [options] list users
```

Output fields:

|  Field        | Description |
|  -----        | ----------- |
|  Name         | User name |
|  Type         | User type |
|  Privileges   | User privileges |
|  Created      | When the user was created |
|  Last Updated | The last time the account password was updated |
|  Last Login   | The last time the user logged in |

Example output:

```
$ maxctrl list users
┌───────┬──────┬────────────┬─────────┬──────────────┬─────────────────────────┐
│ Name  │ Type │ Privileges │ Created │ Last Updated │ Last Login              │
├───────┼──────┼────────────┼─────────┼──────────────┼─────────────────────────┤
│ admin │ inet │ admin      │         │              │ 11/22/2025, 10:37:53 AM │
└───────┴──────┴────────────┴─────────┴──────────────┴─────────────────────────┘
```


#### list commands

List all available module commands.

Usage:

```
maxctrl [options] list commands
```

Output fields:

|  Field    | Description |
|  -----    | ----------- |
|  Module   | Module name |
|  Commands | Available commands |

Example output:

```
$ maxctrl list commands
┌─────────────────┬────────────────────────────────────────────────────────────────────────────┐
│ Module          │ Commands                                                                   |
├─────────────────┼────────────────────────────────────────────────────────────────────────────┤
│ maxscale        │                                                                            |
├─────────────────┼────────────────────────────────────────────────────────────────────────────┤
│ servers         │                                                                            |
├─────────────────┼────────────────────────────────────────────────────────────────────────────┤
│ MariaDBAuth     │                                                                            | 
├─────────────────┼────────────────────────────────────────────────────────────────────────────┤
│ mariadbmon      │ async-create-backup, async-cs-add-node, [...], failover, [...], switchover │
├─────────────────┼────────────────────────────────────────────────────────────────────────────┤
│ MariaDBProtocol │                                                                            │
├─────────────────┼────────────────────────────────────────────────────────────────────────────┤
│ pp_sqlite       │                                                                            │
├─────────────────┼────────────────────────────────────────────────────────────────────────────┤
│ readwritesplit  │ reset-gtid                                                                 │
└─────────────────┴────────────────────────────────────────────────────────────────────────────┘
```

#### list queries

List all active queries being executed through MaxScale. 

In order for this command to work, MaxScale must be configured with `retain_last_statements` set to a value greater than 0.
<!-- TODO: make retain_last_statemens link to that settings description -->

Usage:

```
maxctrl [options] list queries
```

List queries options:
```
  -l, --max-length  Maximum SQL length to display. Use --max-length=0 for no limit.  [number] [default: 120]
```
Output columns:

|  Field    | Description |
| --------- | ----------- |
| Id | Maxscale session id |
| User | Database user |
| Host | Host the user connected from |
| Duration | How long the query has already been running |
| Query | The query text |

Example output:

```
$ maxctrl list queries
┌────┬────────┬──────────────┬──────────┬──────────────────┐
│ Id │ User   │ Host         │ Duration │ Query            │
├────┼────────┼──────────────┼──────────┼──────────────────┤
│ 5  │ myuser │ 10.23.63.100 │ 2s       │ select sleep(60) │
└────┴────────┴──────────────┴──────────┴──────────────────┘
```

### show

#### show server

Show detailed information about a server.
 
The `Parameters` field contains the currently configured parameters for this server. See [`alter server`](#alter-server) for more details about altering server parameters.

Usage:

```
maxctrl [options] show server <server>
```

Output fields:

|  Field               | Description |
|  -----               | ----------- |
|  Server              | Server name |
|  Source              | File where the object is stored in |
|  Address             | Address where the server listens |
|  Port                | The port on which the server listens |
|  State               | Server state |
|  Version             | Server version |
|  Uptime              | Server uptime in seconds |
|  Last Event          | The type of the latest event |
|  Triggered At        | Time when the latest event was triggered at |
|  Services            | Services that use this server |
|  Monitors            | Monitors that monitor this server |
|  Master ID           | The server ID of the master |
|  Node ID             | The node ID of this server |
|  Slave Server IDs    | List of slave server IDs |
|  Current Connections | Current connection count |
|  Total Connections   | Total cumulative connection count |
|  Max Connections     | Maximum number of concurrent connections ever seen |
|  Statistics          | Server statistics |
|  Parameters          | Server parameters |


#### show servers

Show detailed information about all servers.

Usage:

```
maxctrl [options] show servers
```

Output fields are the same as with [`show server`](#show-server) above, just repeated for every server.


#### show service

Show detailed information about a service.   
  
The `Parameters` field contains the currently configured parameters for this service. See [`alter service`](#alter-service) for more details about altering service parameters.

Usage: 

```
maxctrl [options] show service <service>
```

Output Fields:

|  Field               | Description |
|  -----               | ----------- |
|  Service             | Service name |
|  Source              | File where the object is stored in |
|  Router              | Router that the service uses |
|  State               | Service state |
|  Started At          | When the service was started |
|  Users Loaded At     | When the users for the service were loaded |
|  Current Connections | Current connection count |
|  Total Connections   | Total connection count |
|  Max Connections     | Historical maximum connection count |
|  Cluster             | The cluster that the service uses |
|  Servers             | Servers that the service uses |
|  Services            | Services that the service uses |
|  Filters             | Filters that the service uses |
|  Parameters          | Service parameter |
|  Router Diagnostics  | Diagnostics provided by the router module |

#### show services

Show detailed information about all services.

Usage: 

```
maxctrl [options] show services
```

For output details see [`show service`](#show-service) above.



#### show monitor

Show detailed information about a monitor.  
 
The `Parameters` field contains the currently configured parameters for this monitor. See [`alter monitor`](#alter-monitor) for more details about altering monitor parameters.

Usage:

```
maxctrl [options] show monitor <monitor>
```

Output fields:

|  Field               | Description |
|  -----               | ----------- |
|  Monitor             | Monitor name |
|  Source              | File where the object is stored in |
|  Module              | Monitor module |
|  State               | Monitor state |
|  Servers             | The servers that this monitor monitors |
|  Parameters          | Monitor parameters |
|  Monitor Diagnostics | Diagnostics provided by the monitor module |


#### show monitors

Show detailed information about all monitors.

Usage:

```
maxctrl [options]  show monitors
```

For output details see [`show monitor`](#show-monitor) above.


#### show session

Show detailed information about a single session.

The list of sessions can be retrieved with the `list sessions` command. The <session> is the session ID of a particular session.

Usage: 

```
maxctrl [opions] show session <session>
```

Additional Options:
```
--rdns     Perform a reverse DNS lookup on client IPs  [boolean] [default: false]
```

Output Fields:

|  Field             | Description |
|  -----             | ----------- |
|  Id                | Session ID |
|  Service           | The service where the session connected |
|  State             | Session state |
|  User              | Username |
|  Host              | Client host address |
|  Port              | Client network port |
|  Database          | Current default database of the connection |
|  Connected         | Time when the session started |
|  Idle              | How long the session has been idle, in seconds |
|  Parameters        | Session parameters |
|  Client TLS Cipher | Client TLS cipher |
|  Connections       | Ordered list of backend connections |
|  Connection IDs    | Thread IDs for the backend connections |
|  Queries           | Query history |
|  Log               | Per-session log messages |
|  Memory            | Memory usage (not exhaustive) |

#### show sessions

Show detailed information about all sessions.

Usage: 

```
maxctrl [options] show sessions
```  

Additional Options:
```  
  --rdns     Perform a reverse DNS lookup on client IPs  [boolean] [default: false]
```

For output details see [`show session`](#show-session) above.

#### show filter

Show detailed information about a filter.

Usage: 

```
maxctrl [options] show filter <filter>
```


Output fields:

|  Field       | Description |
|  -----       | ----------- |
|  Filter      | Filter name |
|  Source      | File where the object is stored in |
|  Module      | The module that the filter uses |
|  Services    | Services that use the filter |
|  Parameters  | Filter parameters |
|  Diagnostics | Filter diagnostics |


#### show filters

Show detailed information of all filters.

Usage: 

```
maxctrl [options] show filters
```

For output details see [`show filter`](#show-filter) above.


#### show listener

Show detailed information on a listener.

Usage: 
```
maxctrl [options] show listener <listener>
```

Output fields:

|  Field      | Description |
|  -----      | ----------- |
|  Name       | Listener name |
|  Source     | File where the object is stored in |
|  Service    | Services that the listener points to |
|  Parameters | Listener parameters |
|  Diagnostics | Filter diagnostics |

#### show listeners

Show detailed information of all listeners.

Usage:

```
maxscale [options] show filters
```

For output details see [`show listener`](#show-listener) above.


#### show module

This command shows all available parameters as well as detailed version information of a loaded module.

Usage: 
```
maxctrl [options] show module <module>
```


|  Field       | Description |
|  -----       | ----------- |
|  Module      | Module name |
|  Type        | Module type |
|  Version     | Module version |
|  Maturity    | Module maturity |
|  Description | Short description about the module |
|  Parameters  | All the parameters that the module accepts |
|  Commands    | Commands that the module provides |


#### show modules

Displays detailed information about all modules.

```
Usage: show modules
```

For output details see [`show module`](#show-module) above.



#### show maxscale

Show information about the maxscale instance itself. See [`alter maxscale`](#alter-maxscale) for more details about altering MaxScale parameters.


Usage: 
```
maxctrl [options] show maxscale
```


|  Field        | Description |
|  -----        | ----------- |
|  Version      | MaxScale version |
|  Commit       | MaxScale commit ID |
|  Started At   | Time when MaxScale was started |
|  Activated At | Time when MaxScale left passive mode |
|  Uptime       | Time MaxScale has been running |
|  Config Sync  | MaxScale configuration synchronization |
|  Parameters   | Global MaxScale parameters |
|  System       | System Information |


#### show thread

Show detailed information about a worker thread.

Usage: 
```
maxctrl [options] show thread <thread>
```


|  Field                  | Description |
|  -----                  | ----------- |
|  Id                     | Thread ID |
|  State                  | The state of the thread |
|  Accepts                | Number of TCP accepts done by this thread |
|  Reads                  | Number of EPOLLIN events |
|  Writes                 | Number of EPOLLOUT events |
|  Hangups                | Number of EPOLLHUP and EPOLLRDUP events |
|  Errors                 | Number of EPOLLERR events |
|  Avg event queue length | Average number of events returned by one epoll_wait call |
|  Max event queue length | Maximum number of events returned by one epoll_wait call |
|  Max exec time          | The longest time spent processing events returned by a epoll_wait call |
|  Max queue time         | The longest time an event had to wait before it was processed |
|  Current FDs            | Current number of managed file descriptors |
|  Total FDs              | Total number of managed file descriptors |
|  Load (1s)              | Load percentage over the last second |
|  Load (1m)              | Load percentage over the last minute |
|  Load (1h)              | Load percentage over the last hour |
|  QC cache size          | Query classifier size |
|  QC cache inserts       | Number of times a new query was added into the query classification cache |
|  QC cache hits          | How many times a query classification was found in the query classification cache |
|  QC cache misses        | How many times a query classification was not found in the query classification cache |
|  QC cache evictions     | How many times a query classification result was evicted from the query classification cache |
|  Sessions               | The current number of sessions |
|  Zombies                | The current number of zombie connections, waiting to be discarded |
|  Memory                 | The current (partial) memory usage |


#### show threads

Show detailed information about all worker threads.

Usage: 
```
maxctrl [options] show threads
```

Additional options
```
  --kind     The kind of threads to display, only the running or all.  [string] [choices: "running", "all"] [default: "running"]
```


|  Field                  | Description |
|  -----                  | ----------- |
|  Id                     | Thread ID |
|  State                  | The state of the thread |
|  Accepts                | Number of TCP accepts done by this thread |
|  Reads                  | Number of EPOLLIN events |
|  Writes                 | Number of EPOLLOUT events |
|  Hangups                | Number of EPOLLHUP and EPOLLRDUP events |
|  Errors                 | Number of EPOLLERR events |
|  Avg event queue length | Average number of events returned by one epoll_wait call |
|  Max event queue length | Maximum number of events returned by one epoll_wait call |
|  Max exec time          | The longest time spent processing events returned by a epoll_wait call |
|  Max queue time         | The longest time an event had to wait before it was processed |
|  Current FDs            | Current number of managed file descriptors |
|  Total FDs              | Total number of managed file descriptors |
|  Load (1s)              | Load percentage over the last second |
|  Load (1m)              | Load percentage over the last minute |
|  Load (1h)              | Load percentage over the last hour |
|  QC cache size          | Query classifier size |
|  QC cache inserts       | Number of times a new query was added into the query classification cache |
|  QC cache hits          | How many times a query classification was found in the query classification cache |
|  QC cache misses        | How many times a query classification was not found in the query classification cache |
|  QC cache evictions     | How many times a query classification result was evicted from the query classification cache |
|  Sessions               | The current number of sessions |
|  Zombies                | The current number of zombie connections, waiting to be discarded |
|  Memory                 | The current (partial) memory usage |


#### show logging

Show information on current logging configuration.

See [`alter logging`](#alter-logging) for more details about altering logging parameters.

Usage: 
```
maxctrl [options] show logging
```


|  Field              | Description |
|  -----              | ----------- |
|  Current Log File   | The current log file MaxScale is logging into |
|  Enabled Log Levels | List of log levels enabled in MaxScale |
|  Parameters         | Logging parameters |


#### show commands

This command shows the parameters a specific command expects with the parameter descriptions.

For a list of available commands use [`list-commands`](#list-commands).

Usage:
```
maxctrl [options] show commands <module>
```


|  Field        | Description |
|  -----        | ----------- |
|  Command      | Command name |
|  Parameters   | Parameters the command supports |
|  Descriptions | Parameter descriptions |


#### show qc\_cache

Show contents (statement and hits) of query classifier cache.

```
maxctrl [options] show qc_cache
```

Output fields:

| Field | Description |
| ----- | ----------- |
| Statement │ TODO |
| Hits  │ TODO |
│ Count  │ TODO |
│ Avg. Duration  │ TODO |
│ Moving Avg. Duration  │ TODO |
│ Sum Duration  │ TODO |
│ Type  │ TODO |
│ Size  │ TODO |

#### show dbusers

Show information about the database users of the service.

Usage: 
```
maxctrl [options] show dbusers <service>
```


|  Field  | Description |
|  -----  | ----------- |
|  User   | The user name of the account |
|  Host   | The host of the account |
|  Plugin | Authentication plugin |
|  TLS    | Whether TLS is required from this user |
|  Super  | Does the user have a SUPER grant |
|  Global | Does the user have global database access |
|  Proxy  | Whether this is a proxy user |
|  Role   | The default role for this user |


### set

#### set server

Set a servers state.

```
Usage: set server <server> <state>
```

Additional options:
```
  --force  If combined with the `maintenance` state, this forcefully closes all connections to the target server  [boolean] [default: false]
```

If <server> is monitored by a monitor, this command should only be used to set the server into the `maintenance` or the `drain` state. Any other states will be overridden by the monitor on the next monitoring interval. 

To manually control server states, use the [`stop monitor <name>`](#stop-monitor) command to stop the monitor before setting the server states manually.

When a server is set into the `drain` state, no new connections to it are allowed but existing connections are allowed to gracefully close. Servers with the `Master` status cannot be drained or set into maintenance mode. To clear a state set by this command, use the [`clear server`](#clear-server) command.

To forcefully close all connections to a server, use `set server <name> maintenance --force`

<!-- TODO mention all supported <state> values -->

Example output:

```
$ maxctrl [options] set server slave-1 maintenance
OK    
```


### clear

#### clear server

This command clears a server state set by the [`set server <server> <state>`](#set-server) command.

Usage: 
```
clear server <server> <state>
```

<!-- TODO does this command have an extra --force option like its set counterpart? -->

### enable

#### enable log-priority

Enable log output for a specific log priority.

Usage:
```
maxctrl [options] enable log-priority <log>
```

Possible `<log>` values are `warning`, `notice`,`info` and `debug`.  

The `debug` log priority is only available for debug builds of MaxScale.  
There is no `error` priority as such messages are always logged.


### disable

#### disable log-priority

Disable log output for a specific log priority.

Usage:
```  
maxctrl [option] disable log-priority <log>
```

The `debug` log priority is only available for debug builds of MaxScale.  
There is no `error` priority as such messages are always logged.

### create

#### create server

```
Usage: create server <name> <host|socket> [port] [params...]

Create server options:
      --services  Link the created server to these services  [array]
      --monitors  Link the created server to these monitors  [array]


The created server will not be used by any services or monitors unless the --services or --monitors options are given. The list of servers a service or a monitor uses can be altered with the `link` and `unlink` commands. If the <host|socket> argument is an absolute path, the server will use a local UNIX domain socket connection. In this case the [port] argument is ignored.

The recommended way of declaring parameters is with the new `key=value` syntax added in MaxScale 6.2.0. Note that for some parameters (e.g. `extra_port` and `proxy_protocol`) this is the only way to pass them. The redundant option parameters have been deprecated in MaxScale 22.08.
```

#### create monitor

```
Usage: create monitor <name> <module> [params...]

Create monitor options:
      --servers  Link the created monitor to these servers. All non-option arguments after --servers are interpreted as server names e.g. `--servers srv1 srv2 srv3`.  [array]

The list of servers given with the --servers option should not contain any servers that are already monitored by another monitor. The last argument to this command is a list of key=value parameters given as the monitor parameters. The redundant option parameters have been deprecated in MaxScale 22.08.
```

#### create service

```
Usage: service <name> <router> <params...>

Create service options:
      --servers   Link the created service to these servers. All non-option arguments after --servers are interpreted as server names e.g. `--servers srv1 srv2 srv3`.  [array]
      --filters   Link the created service to these filters. All non-option arguments after --filters are interpreted as filter names e.g. `--filters f1 f2 f3`.  [array]
      --services  Link the created service to these services. All non-option arguments after --services are interpreted as service names e.g. `--services svc1 svc2 svc3`.  [array]
      --cluster   Link the created service to this cluster (i.e. a monitor)  [string]

The last argument to this command is a list of key=value parameters given as the service parameters. If the --servers, --services or --filters options are used, they must be defined after the service parameters. The --cluster option is mutually exclusive with the --servers and --services options.

Note that the `user` and `password` parameters must be defined.
```

#### create filter

```
Usage: filter <name> <module> [params...]

The last argument to this command is a list of key=value parameters given as the filter parameters.
```

#### create listener

```
Usage: create listener <service> <name> <port> [params...]


The new listener will be taken into use immediately. The last argument to this command is a list of key=value parameters given as the listener parameters. These parameters override any parameters set via command line options: e.g. using `protocol=mariadb` will override the `--protocol=cdc` option. The redundant option parameters have been deprecated in MaxScale 22.08.
```

#### create user

```
Usage: create user <name> <password>

Create user options:
      --type  Type of user to create  [string] [choices: "admin", "basic"] [default: "basic"]

By default the created user will have read-only privileges. To make the user an administrative user, use the `--type=admin` option. Basic users can only perform `list` and `show` commands.
```

#### create report

```
Usage: create report <file>


The generated report contains the state of all the objects in MaxScale as well as all other required information needed to diagnose problems.
```

### destroy

#### destroy server

```
Usage: destroy server <name>

Destroy options:
      --force  Remove the server from monitors and services before destroying it  [boolean] [default: false]

The server must be unlinked from all services and monitor before it can be destroyed.
```

#### destroy monitor

```
Usage: destroy monitor <name>

Destroy options:
      --force  Remove monitored servers from the monitor before destroying it  [boolean] [default: false]


The monitor must be unlinked from all servers before it can be destroyed.
```

#### destroy listener

```
Usage: destroy listener { <listener> | <service> <listener> }


Destroying a listener closes the listening socket, opening it up for immediate reuse. If only one argument is given and it is the name of a listener, it is unconditionally destroyed. If two arguments are given and they are a service and a listener, the listener is only destroyed if it is for the given service.
```

#### destroy service

```
Usage: destroy service <name>

Destroy options:
      --force  Remove filters, listeners and servers from service before destroying it  [boolean] [default: false]


The service must be unlinked from all servers and filters. All listeners for the service must be destroyed before the service itself can be destroyed.
```

#### destroy filter

```
Usage: destroy filter <name>

Destroy options:
      --force  Automatically remove the filter from all services before destroying it  [boolean] [default: false]


The filter must not be used by any service when it is destroyed.
```

#### destroy user

```
Usage: destroy user <name>


The last remaining administrative user cannot be removed. Create a replacement administrative user before attempting to remove the last administrative user.
```

#### destroy session

```
Usage: destroy session <id>

Destroy options:
      --ttl  Give session this many seconds to gracefully close  [number] [default: 0]


This causes the client session with the given ID to be closed. If the --ttl option is used, the session is given that many seconds to gracefully stop. If no TTL value is given, the session is closed immediately.
```

### link

#### link service

```
Usage: link service <name> <target...>


This command links targets to a service, making them available for any connections that use the service. A target can be a server, another service or a cluster (i.e. a monitor). Before a server is linked to a service, it should be linked to a monitor so that the server state is up to date. Newly linked targets are only available to new connections, existing connections will use the old list of targets. If a monitor (a cluster of servers) is linked to a service, the service must not have any other targets linked to it.
```

#### link monitor

```
Usage: link monitor <name> <server...>


Linking a server to a monitor will add it to the list of servers that are monitored by that monitor. A server can be monitored by only one monitor at a time.
```

### unlink

#### unlink service

```
Usage: unlink service <name> <target...>


This command unlinks targets from a service, removing them from the list of available targets for that service. New connections to the service will not use the unlinked targets but existing connections can still use the targets. A target can be a server, another service or a cluster (a monitor).
```

#### unlink monitor

```
Usage: unlink monitor <name> <server...>


This command unlinks servers from a monitor, removing them from the list of monitored servers. The servers will be left in their current state when they are unlinked from a monitor.
```

### start

#### start service

```
Usage: start service <name>


This starts a service stopped by `stop service <name>`
```

#### start listener

```
Usage: start listener <name>


This starts a listener stopped by `stop listener <name>`
```

#### start monitor

```
Usage: start monitor <name>


This starts a monitor stopped by `stop monitor <name>`
```

#### start services

```
Usage: start [services|maxscale]


This command will execute the `start service` command for all services in MaxScale.
```

### stop

#### stop service

```
Usage: stop service <name>

Stop options:
      --force  Close existing connections after stopping the service  [boolean] [default: false]


Stopping a service will prevent all the listeners for that service from accepting new connections. Existing connections will still be handled normally until they are closed.
```

#### stop listener

```
Usage: stop listener <name>

Stop options:
      --force  Close existing connections after stopping the listener  [boolean] [default: false]


Stopping a listener will prevent it from accepting new connections. Existing connections will still be handled normally until they are closed.
```

#### stop monitor

```
Usage: stop monitor <name>


Stopping a monitor will pause the monitoring of the servers. This can be used to manually control server states with the `set server` command.
```

#### stop services

```
Usage: stop [services|maxscale]

Stop options:
      --force  Close existing connections after stopping all services  [boolean] [default: false]


This command will execute the `stop service` command for all services in MaxScale.
```

### alter

#### alter server

```
Usage: alter server <server> <key=value> ...


To display the server parameters, execute `show server <server>`.
The parameters should be given in the `key=value` format. This command also supports the legacy method
of passing parameters as `key value` pairs but the use of this is not recommended.
```

#### alter monitor

```
Usage: alter monitor <monitor> <key=value> ...


To display the monitor parameters, execute `show monitor <monitor>`
The parameters should be given in the `key=value` format. This command also supports the legacy method
of passing parameters as `key value` pairs but the use of this is not recommended.
```

#### alter service

```
Usage: alter service <service> <key=value> ...


To display the service parameters, execute `show service <service
The parameters should be given in the `key=value` format. This command also supports the legacy method
of passing parameters as `key value` pairs but the use of this is not recommended.
```

#### alter service-filters

```
Usage: alter service-filters <service> [filters...]


The order of the filters given as the second parameter will also be the order in which queries pass through the filter chain. If no filters are given, all existing filters are removed from the service.

For example, the command `maxctrl alter service-filters my-service A B C` will set the filter chain for the service `my-service` so that A gets the query first after which it is passed to B and finally to C. This behavior is the same as if the `filters=A|B|C` parameter was defined for the service.

The parameters should be given in the `key=value` format. This command also supports the legacy method
of passing parameters as `key value` pairs but the use of this is not recommended.
```

#### alter filter

```
Usage: alter filter <filter> <key=value> ...


To display the filter parameters, execute `show filter <filter>`. Some filters support runtime configuration changes to all parameters. Refer to the filter documentation for details on whether it supports runtime configuration changes and which parameters can be altered.

The parameters should be given in the `key=value` format. This command also supports the legacy method
of passing parameters as `key value` pairs but the use of this is not recommended.
Note: To pass options with dashes in them, surround them in both single and double quotes:

      maxctrl alter filter my-namedserverfilter target01 '"->master"'
```

#### alter listener

```
Usage: alter listener <listener> <key=value> ...


To display the listener parameters, execute `show listener <listener>`
The parameters should be given in the `key=value` format. This command also supports the legacy method
of passing parameters as `key value` pairs but the use of this is not recommended.
```

#### alter logging

```
Usage: alter logging <key=value> ...

To display the logging parameters, execute `show logging`
The parameters should be given in the `key=value` format. This command also supports the legacy method
of passing parameters as `key value` pairs but the use of this is not recommended.
```

#### alter maxscale

```
Usage: alter maxscale <key=value> ...


To display the MaxScale parameters, execute `show maxscale`.
The parameters should be given in the `key=value` format. This command also supports the legacy method
of passing parameters as `key value` pairs but the use of this is not recommended.
```

#### alter user

```
Usage: alter user <name> <password>


Changes the password for a user. To change the user type, destroy the user and then create it again.
```

#### alter session

```
Usage: alter session <session> <key=value> ...


Alter parameters of a session. To get the list of modifiable parameters, use `show session <session>`
The parameters should be given in the `key=value` format. This command also supports the legacy method
of passing parameters as `key value` pairs but the use of this is not recommended.
```

#### alter session-filters

```
Usage: alter session-filters <session> [filters...]


The order of the filters given as the second parameter will also be the order in which queries pass through the filter chain. If no filters are given, all existing filters are removed from the session. The syntax is similar to `alter service-filters`.
```

### rotate

#### rotate logs

```
Usage: rotate logs


This command is intended to be used with the `logrotate` command.
```

### reload

#### reload service

```
Usage: reload service <service>

```

#### reload tls

```
Usage: reload tls <service>


This command reloads the TLS certificates for all listeners and servers as well as the REST API in MaxScale. The REST API JWT signature keys are also rotated by this command.
```

#### reload session

```
Usage: reload session <id>


This command reloads the configuration of a session. When a session is reloaded, it internally restarts the MaxScale session. This means that new connections are created and taken into use before the old connections are discarded. The session will use the latest configuration of the service the listener it used pointed to. This means that the behavior of the session can change as a result of a reload if the configuration has changed. If the reloading fails, the old configuration will remain in use. The external session ID of the connection will remain the same as well as any statistics or session level alterations that were done before the reload.
```

#### reload sessions

```
Usage: reload sessions


This command reloads the configuration of all sessions. When a session is reloaded, it internally restarts the MaxScale session. This means that new connections are created and taken into use before the old connections are discarded. The session will use the latest configuration of the service the listener it used pointed to. This means that the behavior of the session can change as a result of a reload if the configuration has changed. If the reloading fails, the old configuration will remain in use. The external session ID of the connection will remain the same as well as any statistics or session level alterations that were done before the reload.
```

### call

#### call command

```
Usage: call command <module> <command> [params...]


To inspect the list of module commands, execute `list commands`
```

### api

#### api get

```
Usage: get <resource> [path]

API options:
      --sum     Calculate sum of API result. Only works for arrays of numbers e.g. `api get --sum servers data[].attributes.statistics.connections`.  [boolean] [default: false]
      --pretty  Pretty-print output.  [boolean] [default: false]

Perform a raw REST API call. The path definition uses JavaScript syntax to extract values. For example, the following command extracts all server states as an array of JSON values: maxctrl api get servers data[].attributes.state
```

#### api post

```
Usage: post <resource> <value>

API options:
      --sum     Calculate sum of API result. Only works for arrays of numbers e.g. `api get --sum servers data[].attributes.statistics.connections`.  [boolean] [default: false]
      --pretty  Pretty-print output.  [boolean] [default: false]

Perform a raw REST API call. The provided value is passed as-is to the REST API after building it with JSON.parse
```

#### api patch

```
Usage: patch <resource> [path]

API options:
      --sum     Calculate sum of API result. Only works for arrays of numbers e.g. `api get --sum servers data[].attributes.statistics.connections`.  [boolean] [default: false]
      --pretty  Pretty-print output.  [boolean] [default: false]

Perform a raw REST API call. The provided value is passed as-is to the REST API after building it with JSON.parse
```

### classify

Classify SQL statement using MaxScale and display the result.

Usage:

```
maxctrl [options] classify "...statement..."
```
Output fields:

| Field | Description |
| ----- | ----------- |
| Parse result | See list below |
| Type mask | See list below |
| Operation | See list below |
| Has where clause | Not functional yet? |
| Fields | Table columns used in the statement |
| Functions | Functions and operators used in the statement |
| Canonocal | Canonical form of the statement with all constant values being replaced with `?` placeholders |

Parse result values:

| Parser result | Description |
| ------------- | ----------- |
| Parser::Result::INVALID          | The query was not recognized or could not be parsed. |
| Parser::Result::TOKENIZED        | The query was classified based on tokens; incompletely classified. |
| Parser::Result::PARTIALLY_PARSED | The query was only partially parsed; incompletely classified. |
| Parser::Result::PARSED           | The query was fully parsed; completely classified. |
 
Type mask values:

| Type mask                    |  Description |
| ---------------------------- | ------------ |
| sql::TYPE_READ               |  Read database data:any |
| sql::TYPE_WRITE              |  Master data will be  modified:master |
| sql::TYPE_MASTER_READ        |  Read from the master:master |
| sql::TYPE_SESSION_WRITE      |  Session data will be modified:master or all |
| sql::TYPE_USERVAR_WRITE      |  Write a user variable:master or all |
| sql::TYPE_USERVAR_READ       |  Read a user variable:master or any |
| sql::TYPE_SYSVAR_READ        |  Read a system variable:master or any |
| sql::TYPE_GSYSVAR_READ       |  Read global system variable:master or any |
| sql::TYPE_GSYSVAR_WRITE      |  Write global system variable:master or all |
| sql::TYPE_BEGIN_TRX          |  BEGIN or START TRANSACTION |
| sql::TYPE_ENABLE_AUTOCOMMIT  |  SET autocommit=1 |
| sql::TYPE_DISABLE_AUTOCOMMIT |  SET autocommit=0 |
| sql::TYPE_ROLLBACK           |  ROLLBACK |
| sql::TYPE_COMMIT             |  COMMIT |
| sql::TYPE_PREPARE_NAMED_STMT |  Prepared stmt with name from user:all |
| sql::TYPE_PREPARE_STMT       |  Prepared stmt with id provided by server:all |
| sql::TYPE_EXEC_STMT          |  Execute prepared statement:master or any |
| sql::TYPE_CREATE_TMP_TABLE   |  Create temporary table:master (could be all) |
| sql::TYPE_DEALLOC_PREPARE    |  Dealloc named prepare stmt:all |
| sql::TYPE_READONLY           |  The READ ONLY part of SET TRANSACTION |
| sql::TYPE_READWRITE          |  The READ WRITE part of SET TRANSACTION  |
| sql::TYPE_NEXT_TRX           |  SET TRANSACTION that's only for the next transaction |


Operation values:

| Operation |
| --------- |
| sql::OP_UNDEFINED |
| sql::OP_ALTER |
| sql::OP_ALTER_TABLE |
| sql::OP_CALL |
| sql::OP_CHANGE_DB |
| sql::OP_CREATE |
| sql::OP_CREATE_ROLE |
| sql::OP_CREATE_TABLE |
| sql::OP_CREATE_USER |
| sql::OP_DELETE |
| sql::OP_DROP |
| sql::OP_DROP_TABLE |
| sql::OP_EXECUTE |
| sql::OP_EXPLAIN |
| sql::OP_GRANT |
| sql::OP_INSERT |
| sql::OP_KILL |
| sql::OP_LOAD |
| sql::OP_LOAD_LOCAL |
| sql::OP_REVOKE |
| sql::OP_SELECT |
| sql::OP_SET |
| sql::OP_SET_TRANSACTION |
| sql::OP_SHOW |
| sql::OP_SHOW_DATABASES |
| sql::OP_TRUNCATE |
| sql::OP_UPDATE |

Example output:

```
$  maxctrl classify "select user, host as foo from mysql.user where 2 = 1 + 1"
┌──────────────────┬──────────────────────────────────────────────────────────┐
│ Parse result     │ Parser::Result::PARSED                                   │
├──────────────────┼──────────────────────────────────────────────────────────┤
│ Type mask        │ sql::TYPE_READ                                           │
├──────────────────┼──────────────────────────────────────────────────────────┤
│ Operation        │ sql::OP_SELECT                                           │
├──────────────────┼──────────────────────────────────────────────────────────┤
│ Has where clause │                                                          │
├──────────────────┼──────────────────────────────────────────────────────────┤
│ Fields           │ user                                                     │
│                  │ host                                                     │
├──────────────────┼──────────────────────────────────────────────────────────┤
│ Functions        │ =: ()                                                    │
│                  │ +: ()                                                    │
├──────────────────┼──────────────────────────────────────────────────────────┤
│ Canonical        │ select user, host as foo from mysql.user where ? = ? + ? │
└──────────────────┴──────────────────────────────────────────────────────────┘
```



<sub>_This page is licensed: CC BY-SA / Gnu FDL_</sub>

{% @marketo/form formId="4316" %}
