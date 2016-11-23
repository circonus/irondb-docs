# Installation

# snowth

Distributed time-series storage and analytics system

## Build Instructions

For building snowth on an omnios6 box (like this [Vagrant VM](https://atlas.hashicorp.com/omniti/boxes/omnios-r151006)), see:

* `omnios-install-dependencies.sh`

* `omnios-build.sh`

## Getting Started

To setup a minimal node of a multi-node cluster use

 NODES=2 ./scratch.sh -D

NODES defaults 1 and integral values up to and including 6 are supported. Note that this only starts a single node.

This will roughly do the following steps:

1. Create a valid `snowth.conf`. Use the provided snowth.conf file as an example to get started.

2. Create a valid topology `snowth.topo`. The topology specifies the nodes in the cluster.

 Use `topology.sample` to get started. A minimal `snowth-topo` could look like this:

 <nodes>

 <node id="bb6f7162-4828-11df-bab8-6bac200dcc2a"

 address="127.0.0.1"

 apiport="8112"

 port="9112"

 weight="128"/>

 </nodes>

3. Compile the topology with:

 ./snowthimport -c snowth.conf -i snowth.topology

 This will create a file named like a has key, e.g. `e9968ea18b2793f168a6c67af6f119108c088fe6023d6b8940a495fd4acb1538`

 that contains the complied topology.

4. Update the topology stanza in `snowth.conf` to point at the created topology file, e.g.

 <topology path="/opt/circonus/etc/snowth-topo/" active="e9968ea18b2793f168a6c67af6f119108c088fe6023d6b8940a495fd4acb1538" redo="/var/tmp/tredo/{node}"/>

5. Run `snowthd`, e.g with `./snowthd -u nobody -g nobody -D -c snowth.conf`. See `snowthd -h` for instructions.

## Documentation

An API manual can be found in /docs. Use make to generate a pdf

version from the doxygen xml files.

Documentation about lua extensions is available at: `http://$HOST:8112/#extensions`.

## Running LUA profiler

### Install dependency luajit-2.1

```
cd /opt

git clone http://luajit.org/git/luajit-2.0.git

mv luajit-2.0 luajit

cd luajit

git checkout v2.1

make -j

sudo make install

# test your new luajit:

luajit-2.1.0-alpha -v
```

### Run luajit proifler

```
pathToSnowth=/opt/snowth

luajitCmd=/usr/local/bin/luajit-2.1.0-alpha

cd $pathToSnowth/lua/test

packagePath=`$luajitCmd -e "print(package.path)"`";../support/?.lua;../extension/?.lua;./?.lua"

echo "Setting LUA_PATH to $packagePath"

# Run tests with profiler (-jp)

LUA_PATH=$packagePath $luajitCmd -jp -l test_runner -e 'test_runner.run()'
```
