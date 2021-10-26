Statoshi: Bitcoin Core + statistics logging

What is Statoshi?
----------------

Statoshi's objective is to protect Bitcoin by bringing transparency to the activity
occurring on the node network. By making this data available, Bitcoin developers can
learn more about node performance, gain operational insight about the network, and
the community can be informed about attacks and aberrant behavior in a timely fashion.

There is a live Grafana dashboard at [statoshi.info](https://statoshi.info)

License
-------

Statoshi is released under the terms of the MIT license. See [COPYING](COPYING) for more
information or see http://opensource.org/licenses/MIT.

Development Process
-------

Statoshi's changeset to Bitcoin Core is applied in the `master` branch and is
built and tested after each merge from upstream or from a pull request. However,
it not guaranteed to be completely stable. We do not recommend using Statoshi
as a Bitcoin wallet.

A guide for Statoshi developers is available [here](https://blog.lopp.net/statoshi-developer-s-guide/)

Other Notes
-------

A system metrics daemon is available [here](https://github.com/jlopp/bitcoin-utils/blob/master/systemMetricsDaemon.py)

Statoshi also supports running multiple nodes that emit metrics to a single graphite instance.
In order to facilitate this, you can add a line to bitcoin.conf that will partition each
metric by the name of the host: statshostname=yourNodeName
--
## Docker Notes & [DigitalOcean.com](https://m.do.co/c/ae5c7d05da91)

## [stats.bitcoincore.dev](https://github.com/bitcoincore-dev/stats.bitcoincore.dev/packages/314536)

#### Full Build

This runs the statoshi configuration and [displays statsd data from the pruned node](http://stats.bitcoincore.dev:3000).

```
docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/stats.bitcoincore.dev
0.20.99             a27f8eb4ad39        4 minutes ago       2.98GB
```

Pull image from the command line:

	docker pull docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/stats.bitcoincore.dev:0.20.99

Use as base image in DockerFile:

	FROM docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/stats.bitcoincore.dev:0.20.99


## [stats.bitcoincore.dev.slim](https://github.com/bitcoincore-dev/stats.bitcoincore.dev/packages/315130)

#### Slim Build

This runs the slim (precompiled signed binaries) configuration and [displays statsd data from the pruned node](http://stats.bitcoincore.dev:3000).

```
docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/stats.bitcoincore.dev.slim
0.20.99             390876b14625        24 minutes ago      1.63GB
```
Pull image from the command line:

	docker pull docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/stats.bitcoincore.dev.slim:0.20.99

Use as base image in DockerFile:

	FROM docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/stats.bitcoincore.dev.slim:0.20.99

## [stats.bitcoincore.dev.gui](https://github.com/bitcoincore-dev/stats.bitcoincore.dev/packages/315116)

#### Gui Build

This runs the gui [(graphite/grafana) configuration](http://stats.bitcoincore.dev:3000) and pulls data from
[http://stats.bitcoincore.dev:8080](http://stats.bitcoincore.dev:8080). Useful as a demo or if you don't want to run your own statoshi instance.

```
docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/stats.bitcoincore.dev.gui
0.20.99             737a8acf33c5        About an hour ago   1.23GB
```

Pull image from the command line:

	docker pull docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/stats.bitcoincore.dev.gui:0.20.99
	
Use as base image in DockerFile:

	FROM docker.pkg.github.com/bitcoincore-dev/stats.bitcoincore.dev/stats.bitcoincore.dev.gui:0.20.99

## make help


##### $ <code>make</code>
make[1]: Entering directory '/home/runner/work/statoshi.dev/statoshi.dev'

	[ARGUMENTS]	
      args:
        - HOME=/home/runner
        - PWD=/home/runner/work/statoshi.dev/statoshi.dev
        - UMBREL=false
        - THIS_FILE=GNUmakefile
        - TIME=1635287295
        - HOST_USER=runner
        - HOST_UID=0
        - PUBLIC_PORT=80
        - NODE_PORT=8333
        - SERVICE_TARGET=shell
        - ALPINE_VERSION=3.11.6
        - WHISPER_VERSION=1.1.7
        - CARBON_VERSION=1.1.7
        - GRAPHITE_VERSION=1.1.7
        - STATSD_VERSION=0.8.6
        - GRAFANA_VERSION=7.0.0
        - DJANGO_VERSION=2.2.24
        - PROJECT_NAME=statoshi.dev
        - DOCKER_BUILD_TYPE=all
        - SLIM=false
        - DOCKERFILE=statoshi.dev
        - DOCKERFILE_BODY=docker/statoshi.all
        - GIT_USER_NAME=
        - GIT_USER_EMAIL=
        - GIT_SERVER=https://github.com
        - GIT_PROFILE=randymcmillan
        - GIT_BRANCH=master
        - GIT_HASH=2fae8a7b1e3ee0f3af70fa825333d2d390c7ddae
        - GIT_REPO_ORIGIN=https://github.com/RandyMcMillan/statoshi.dev
        - GIT_REPO_NAME=statoshi.dev
        - GIT_REPO_PATH=/home/runner/statoshi.dev
        - DOCKERFILE=statoshi.dev
        - DOCKERFILE_PATH=/home/runner/statoshi.dev/statoshi.dev
        - BITCOIN_CONF=/home/runner/statoshi.dev/conf/bitcoin.conf
        - BITCOIN_DATA_DIR=/home/runner/.bitcoin
        - STATOSHI_DATA_DIR=/home/runner/.statoshi
        - NOCACHE=
        - VERBOSE=
        - PUBLIC_PORT=80
        - NODE_PORT=8333
        - PASSWORD=changeme
        - CMD_ARGUMENTS=

	[USAGE]:	make [BUILD] run [EXTRA_ARGUMENTS]	

		make init header build run user=root uid=0 nocache=false verbose=true

	[DEV ENVIRONMENT]:	

		make header user=root
		make shell  user=root
		make shell  user=runner

	[EXTRA_ARGUMENTS]:	set build variables	

		nocache=true
		            	add --no-cache to docker command and apk add 
		port=integer
		            	set PUBLIC_PORT default 80

		nodeport=integer
		            	set NODE_PORT default 8333

		            	TODO

	[DOCKER COMMANDS]:	push a command to the container	

		cmd=command 	
		cmd="command"	
		             	send CMD_ARGUMENTS to the [TARGET]

	[EXAMPLE]:

		make all run user=root uid=0 no-cache=true verbose=true
		make report all run user=root uid=0 no-cache=true verbose=true cmd="top"
		make a r port=80 no-cache=true verbose=true cmd="ls"

	[COMMAND_LINE]:

	stats-console              # container command line
	stats-bitcoind             # start container bitcoind -daemon
	stats-debug                # container debug.log output
	stats-whatami              # container OS profile

	stats-cli -getmininginfo   # report mining info
	stats-cli -gettxoutsetinfo # report txo info

	#### WARNING: (effects host datadir) ####
	
	stats-prune                # default in bitcoin.conf is prune=1 - start pruning node
	
make[1]: Leaving directory '/home/runner/work/statoshi.dev/statoshi.dev'
