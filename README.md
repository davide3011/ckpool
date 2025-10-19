# CKPOOL + CKPROXY + libckpool

**by Con Kolivas**

Ultra low overhead massively scalable multi-process, multi-threaded modular bitcoin mining pool, proxy, passthrough, and library in C for Linux.

CKPOOL is code provided free of charge under the GPLv3 license but its development is mostly paid for by commissioned funding, and the pool by default contributes 0.5% of solved blocks in pool mode to the development team. Please consider leaving this contribution in the code if you are running it on a pool or contributing to the authors listed in AUTHORS if you use this code to aid funding further development.

## License

GNU Public license V3. See included COPYING for details.

## Bitcoind & Solo Mining Download and Quickstart

```bash
wget https://bitbucket.org/ckolivas/ckpool/raw/master/scripts/install-ckpool-solo.sh
chmod +x install-ckpool-solo.sh
sudo ./install-ckpool-solo.sh
```

## Design

### Architecture

- Low level hand coded architecture relying on minimal outside libraries beyond basic glibc functions for maximum flexibility and minimal overhead that can be built and deployed on any Linux installation.

- Multiprocess+multithreaded design to scale to massive deployments and capitalise on modern multicore/multithread CPU designs.

- Minimal memory overhead.

- Utilises ultra reliable unix sockets for communication with dependent processes.

- Modular code design to streamline further development.

- Standalone library code that can be utilised independently of ckpool.

- Same code can be deployed in many different modes designed to talk to each other on the same machine, local lan or remote internet locations.

### Modes of Deployment

- Simple pool.

- Simple pool with per-user solo mining.

- Simple proxy without the limitations of hashrate inherent in other proxy solutions when talking to ckpool.

- Passthrough node(s) that combine connections to a single socket which can be used to scale to millions of clients and allow the main pool to be isolated from direct communication with clients.

- Library for use by other software.

### Features

- Bitcoind communication to unmodified bitcoind with multiple failover to local or remote locations.

- Local pool instance worker limited only by operating system resources and can be made virtually limitless through use of multiple downstream passthrough nodes.

- Proxy and passthrough modes can set up multiple failover upstream pools.

- Optional share logging.

- Virtually seamless restarts for upgrades through socket handover from exiting instances to new starting instance.

- Configurable custom coinbase signature.

- Configurable instant starting and minimum difficulty.

- Rapid vardiff adjustment with stable unlimited maximum difficulty handling.

- New work generation on block changes incorporate full bitcoind transaction set without delay or requiring to send transactionless work to miners thereby providing the best bitcoin network support and rewarding miners with the most transaction fees.

- Event driven communication based on communication readiness preventing slow communicating clients from delaying low latency ones.

- Stratum messaging system to running clients.

- Accurate pool and per client statistics.

- Multiple named instances can be run concurrently on the same machine.

## Building

Also requires autoconf, automake, and pkgconf:

```bash
sudo apt-get install build-essential yasm autoconf automake libtool libzmq3-dev pkgconf
./autogen.sh
./configure
make
```

### Generated Binaries

Binaries will be built in the `src/` subdirectory. Binaries generated will be:

- **ckpool** - The main pool back end
- **ckproxy** - A link to ckpool that automatically starts it in proxy mode
- **ckpmsg** - An application for passing messages in libckpool format to ckpool
- **notifier** - An application designed to be run with bitcoind's `-blocknotify` to notify ckpool of block changes.

### Installation

Installation is NOT required and ckpool can be run directly from the directory it's built in but it can be installed with:

```bash
sudo make install
```

## Running

ckpool supports the following options:

### Command Line Options

| Option | Long Option | Description |
|--------|-------------|-------------|
| `-B` | `--btcsolo` | Start ckpool in BTCSOLO mode for solo mining |
| `-c CONFIG` | `--config CONFIG` | Override default configuration filename |
| `-g GROUP` | `--group GROUP` | Start ckpool as the specified group ID |
| `-H` | `--handover` | Attempt to receive handover from running instance |
| `-h` | `--help` | Display help |
| `-k` | `--killold` | Shut down existing instance with same name |
| `-L` | `--log-shares` | Log per share information in logs directory |
| `-l LOGLEVEL` | `--loglevel LOGLEVEL` | Change log level (default 5, max debug 7) |
| `-N` | `--node` | Start in passthrough node mode |
| `-n NAME` | `--name NAME` | Change process name |
| `-P` | `--passthrough` | Start in passthrough proxy mode |
| `-p` | `--proxy` | Start in proxy mode |
| `-R` | `--redirector` | Start in redirector mode |
| `-s SOCKDIR` | `--sockdir SOCKDIR` | Specify socket directory (default /tmp) |
| `-u` | `--userproxy` | Start in userproxy mode |

### Mode Descriptions

- **`-B` (BTCSOLO mode)**: Designed for solo mining. All usernames must be valid bitcoin addresses, and 100% of the block reward goes to the user solving the block, minus any donation set.

- **`-c <CONFIG>`**: Override default configuration. Default configs: `ckpool.conf`, `ckproxy.conf`, `ckpassthrough.conf`, `ckredirector.conf`.

- **`-H` (Handover)**: Take client listening socket from running instance and shut it down.

- **`-k` (Kill old)**: Shut down existing instance, otherwise ckpool refuses to start if same name instance is running.

- **`-L` (Log shares)**: Log per share information divided by block height and workbase.

- **`-N` (Node mode)**: Passthrough node with local bitcoind that can submit blocks and monitor hashrate.

- **`-P` (Passthrough)**: Collates incoming connections on single upstream connection while retaining individual presence.

- **`-p` (Proxy)**: Appears as local pool while presenting shares as single user to upstream pool.

- **`-R` (Redirector)**: Filters users that never contribute shares, redirects active users to configured pools.

- **`-u` (Userproxy)**: Proxy mode that accepts username/passwords and opens additional upstream connections.

**Note**: `ckpmsg` and `notifier` support the `-n`, `-p` and `-s` options.

## Configuration

At least one bitcoind is mandatory in ckpool mode with minimum requirements of `server`, `rpcuser` and `rpcpassword` set.

Ckpool takes a JSON encoded configuration file (`ckpool.conf` by default, or `ckproxy.conf` in proxy/passthrough mode unless specified with `-c`). Sample configurations are included with the source.

### Configuration Options

| Option | Type | Description |
|--------|------|-------------|
| `btcd` | Array | Bitcoind configuration with `url`, `auth`, `pass`, and optional `notify` |
| `proxy` | Array | Upstream pool configuration (proxy/passthrough mode) |
| `btcaddress` | String | Bitcoin address for block generation (ignored in BTCSOLO) |
| `btcsig` | String | Optional coinbase signature |
| `blockpoll` | Integer | Block check frequency in ms (default 100) |
| `donation` | Float | Block reward donation percentage (default 0) |
| `nodeserver` | Array | Additional IPs/ports for node communications |
| `nonce1length` | Integer | Extranonce1 length, 2-8 (default 4) |
| `nonce2length` | Integer | Extranonce2 length, 2-8 (default 8) |
| `update_interval` | Integer | Stratum update frequency in seconds (default 30) |
| `version_mask` | String | Valid client version bits as hex (default "1fffe000") |
| `serverurl` | Array | IP(s) to bind to (default all interfaces, port 3333/3334) |
| `redirecturl` | Array | URLs for redirector mode |
| `mindiff` | Integer | Minimum vardiff (default 1) |
| `startdiff` | Integer | Starting difficulty for new clients (default 42) |
| `maxdiff` | Integer | Maximum vardiff (0 = no maximum) |
| `logdir` | String | Log directory (default "logs") |
| `maxclients` | Integer | Maximum client connections |
| `zmqblock` | String | ZMQ blockhash notification interface (default "tcp://127.0.0.1:28332") |

### Default Configuration

If no `btcd` is specified, ckpool looks for bitcoind on `localhost:8332` with username "user" and password "pass".

Entries after valid JSON are ignored and can be used for comments.