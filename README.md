
# docker-bitcoind

[![Docker Stars](https://img.shields.io/docker/stars/jamesob/bitcoind.svg)](https://hub.docker.com/r/jamesob/bitcoind/)
[![Docker Pulls](https://img.shields.io/docker/pulls/jamesob/bitcoind.svg)](https://hub.docker.com/r/jamesob/bitcoind/)

A Docker configuration with sane defaults for running a fully-validating
Bitcoin node. Binaries are retrieved from bitcoincore.org and verified for integrity
based on [the process described here](https://bitcoincore.org/en/download/).

## Tags available

- 0.13.0
- 0.13.1
- 0.13.2
- 0.14.3
- 0.15.2
- 0.16.3
- 0.17.0
- 0.17.0.1
- 0.17.1
- 0.17.2
- 0.18.0
- 0.18.1
- 0.19.0
- 0.19.1
- 0.20.0
- 0.20.1 (latest)

## Quick start

Requires that [Docker be installed](https://docs.docker.com/install/) on the host machine.

```sh
# Create some directory where your bitcoin data will be stored.
$ mkdir /home/youruser/bitcoin_data

$ docker run --name bitcoind -d \
   -e 'BTC_RPCUSER=foo' \
   -e 'BTC_RPCPASSWORD=password' \
   -e 'BTC_TXINDEX=1' \
   -v /home/youruser/bitcoin_data:/bitcoin/data \
   -p 127.0.0.1:8332:8332 \
   --publish 8333:8333 \
   jamesob/bitcoind:0.20.1

$ docker logs -f bitcoind
[ ... ]
```

If you want to use a preexisting data directory and your own config file, run

```sh
$ docker run --name jamesob/bitcoind:0.20.1 -d \
   -v /home/youruser/bitcoin_data:/bitcoin/data \
   -v /home/youruser/bitcoin.conf:/bitcoin/bitcoin.conf \
   -p 127.0.0.1:8332:8332 \
   --publish 8333:8333 \
   jamesob/bitcoind:0.20.1
```

### Building yourself

By default, the container runs under UID,GID=1000 to avoid executing as a privileged
user. If you want to rebuild the container with different settings, you can do so:

```
$ git clone https://github.com/jamesob/docker-bitcoind
$ cd docker-bitcoind
$ docker build -t $YOUR_USER/bitcoind:$SOME_VERSION \
   --build-arg UID=$(id -u) \
   --build-arg GID=$(id -g) \
   --build-arg VERSION=$SOME_VERSION \
   .
```

## Possible volume mounts

| Path | Description |
| ---- | ------- |
| `/bitcoin/data` | Bitcoin's data directory |
| `/bitcoin/bitcoin.conf` | Bitcoin's configuration file |



## Configuration

A custom `bitcoin.conf` file can be placed at `/bitcoin.conf`.
Otherwise, a default will be automatically generated based
on environment variables passed to the container:

| name | default |
| ---- | ------- |
| BTC_RPCUSER | btc |
| BTC_RPCPASSWORD | changemeplz |
| BTC_RPCPORT | 8332 |
| BTC_RPCALLOWIP | ::/0 |
| BTC_RPCCLIENTTIMEOUT | 30 |
| BTC_DISABLEWALLET | 1 |
| BTC_TXINDEX | 0 |
| BTC_TESTNET | 0 |
| BTC_DBCACHE | 0 |
| BTC_ZMQPUBHASHTX | tcp://0.0.0.0:28333 |
| BTC_ZMQPUBHASHBLOCK | tcp://0.0.0.0:28333 |
| BTC_ZMQPUBRAWTX | tcp://0.0.0.0:28333 |
| BTC_ZMQPUBRAWBLOCK | tcp://0.0.0.0:28333 |


## Daemonizing

The smart thing to do if you're daemonizing is to use Docker's [builtin
restart
policies](https://docs.docker.com/config/containers/start-containers-automatically/#use-a-restart-policy),
but if you're insistent on using systemd, you could do something like

```bash
$ cat /etc/systemd/system/bitcoind.service

# bitcoind.service #######################################################################
[Unit]
Description=Bitcoind
After=docker.service
Requires=docker.service

[Service]
ExecStartPre=-/usr/bin/docker kill bitcoind
ExecStartPre=-/usr/bin/docker rm bitcoind
ExecStartPre=/usr/bin/docker pull jamesob/bitcoind
ExecStart=/usr/bin/docker run \
    --name bitcoind \
    -p 8333:8333 \
    -p 127.0.0.1:8332:8332 \
    -v /data/bitcoind:/root/.bitcoin \
    jamesob/bitcoind
ExecStop=/usr/bin/docker stop bitcoind
```

to ensure that bitcoind continues to run.


## Alternatives

- [docker-bitcoind](https://github.com/kylemanna/docker-bitcoind): sort of the
  basis for this repo, but configuration is a bit more confusing.
- [wwcr/bitcoin-core](https://hub.docker.com/r/wwcr/bitcoin-core/dockerfile): trustworthy
  and simple.
