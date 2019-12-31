# Setup OVS
## Starts OVS
First time ovsdb creation (or cleans up existing config)
```
$ mkdir -p /usr/local/etc/openvswitch
$ mkdir -p /usr/local/var/run/openvswitch
$ rm /usr/local/etc/openvswitch/conf.db
$ ovsdb-tool create /usr/local/etc/openvswitch/conf.db /usr/local/share/openvswitch/vswitch.ovsschema
```

 Starts ovsdb-server without SSL
```
$ ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock \
    --remote=db:Open_vSwitch,Open_vSwitch,manager_options \
    --pidfile --detach
```

 Initializes ovsdb
```
$ ovs-vsctl --no-wait init
```

 Starts ovs-vswitchd
```
$ export DB_SOCK=/usr/local/var/run/openvswitch/db.sock
$ ovs-vswitchd unix:$DB_SOCK --pidfile --log-file --detach
```

## Adds bridges
Starts a bridge with kernel datapath
```
$ ovs-vsctl -- add-br br0 -- set Bridge br0 protocols=OpenFlow10,OpenFlow11,OpenFlow12,OpenFlow13,OpenFlow14,OpenFlow15 fail-mode=secure
```

Starts a bridge with userspace datapth
```
$ ovs-vsctl -- add-br br0 -- set Bridge br0 datapath_type=netdev
```

Adds flow
```
$ ovs-ofctl add-flow br0 "actions=NORMAL"
```

Checks the setup
```
$ ovs-vsctl show
$ ovs-ofctl show br0
```

## Tear down setup
```
$ /usr/local/share/openvswitch/scripts/ovs-ctl stop
```

# Testing with namespace

## Using veth
This following commands adds two namespaces for testing.
```
$ ip netns add ns0
$ ip netns add ns1
```

Sets up p0 in ns0
```
$ ip link add p0 type veth peer name ovs-p0
$ ip link set p0 netns ns0
$ ip link set dev ovs-p0 up
$ ovs-vsctl add-port br0 ovs-p0 -- set interface ovs-p0 ofport_request=1

$ ip netns exec ns0 bash << NS_EXEC_EOF
$ ip addr add 192.168.1.2/24 dev p0
$ ip link set dev p0 up
$ NS_EXEC_EOF
```

Set up p1 in ns1
```
$ ip link add p1 type veth peer name ovs-p1
$ ip link set p1 netns ns1
$ ip link set dev ovs-p1 up
$ ovs-vsctl add-port br0 ovs-p1 -- set interface ovs-p0 ofport_request=2

$ ip netns exec ns1 bash << NS_EXEC_EOF
$ ip addr add 192.168.1.3/24 dev p1
$ ip link set dev p1 up
$ NS_EXEC_EOF
```

## Using tap
```
$ ip netns add ns0
$ ovs-vsctl add-port br0 int0 -- set Interface int0 type=internal
$ ip link set int0 netns ns0
```

Bring the tap device up in ns0
```
$ ip netns exec ns0 bash
$ ip addr add 192.168.1.2/24 dev int0
$ ip route add default via 192.168.1.2
$ ip link set int0 up
```

Test with main name space
```
$ ip addr add 192.168.1.1/24 dev br0
$ ip link set br0 up
$ ping 192.168.1.2
```

## Tear down namespace setup
In main namespace
```
$ ovs-vsctl del-port br0 ovs-p0
$ ip link del ovs-p0
$ ip netns del ns0
```
