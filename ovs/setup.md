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
