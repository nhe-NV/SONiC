# Ip Interface Loopback Action Test Plan

## Related documents

| **Document Name** | **Link** |
|-------------------|----------|
| rif_loopback_action_hld | [https://github.com/liorghub/SONiC/blob/rif_loopback_action/doc/rif-loopback-action/rif_loopback_action.md]|


## Overview
RIF loopback action is a feature which allows user to change the way router handles routed packets with egress port equal to ingress port.

1. When RIF loopback action is configured to drop, routed packet ingress port and egress port can not be equal. In other words, if routed packet entered the router from a port and destined to egress from the same one, it will be dropped.
2. When RIF loopback action is configured to forward, routed packet ingress port and egress port can be equal.
3. Default loopback action in SAI is forward. SONiC will rely on SAI default (the motivation here is not to change the behaviour if other SAI vendors have different default).


## Requirements

#### The Rif loopback action is support on following ip interface:
1. Ethernet interface
2. Vlan interface
3. PortChannel interface

#### This feature will support the following commands:

1. config: set RIF loopback action
2. show: display loopback action of all RIFs

#### This feature will provide error handling for the next situations:

1. Invalid object reference
2. Invalid options/parameters

### Scope

The test is to verify the loopback action can be configured, and the loopback traffic can be forwarded/droppped as expect according to the action configured on the ip interface.   

Run the fully regression with loopback action set to the drop to make sure no other feature will be broken.

### Scale / Performance

No scale test related

### Related **DUT** CLI commands

#### Config
The following command can be used to configure loopback action on L3 interface:
```
config interface loopback-action <interface-name> <action>
```
action options: "drop", "forward"

Examples:
```
config interface loopback-action Ethernet248 drop
config interface loopback-action Vlan100 forward
config interface loopback-action PortChannel1 drop
```

#### Show
The following command can be used to show loopback action:
```
show ip interfaces loopback action 
```
Example:
```
show ip interfaces loopback-action
Interface     Action      
------------  ----------  
Ethernet248   drop     
Vlan100       forward     
```
### Related DUT configuration files

```
"INTERFACE": {
    "Ethernet248": {
        "loopback_action": "drop"
    },
},
"PORTCHANNEL_INTERFACE": {
    "PortChannel1": {
        "loopback_action": "drop"
    },
},
"VLAN_INTERFACE": {
    "Vlan100": {
        "loopback_action": "drop"
    },
}
```
### Supported topology
The test will be supported on any topology


## Test cases

### Test cases #1 - Verify the loopback action can be configured and worked as expect
1. Configure the loopback action to drop on the interface with cli command
2. Verify the the loopback action is configured correctly with show cli command.
3. Send traffic to do the validation: 
   - Verify the traffic will be dropped as expected.
   - The RIF drop counter will increase as expected
4. Change it back to forward
5. Verify the the loopback action is configured correctly with show cli command.
6. Send traffic to do the validation
   - Verify the traffic will be dropped as expected.
   - The RIF drop counter will increase as expected
7. Recover the loopback action to original one.
   
### Test cases #2 - Disable/Enable the interface and check if the loopback action still work
1. Configure the loopback action on the interface, some to drop, some to forward.
2. Disable the interfaces
3. Enable the interfaces back
4. Verify the loopback action is not changed with show cli command.
5. Send traffic to do the validation: 
   - Verify the traffic will be dropped/forwarded as expected.
   - The RIF drop counter will increase or not increased as expected.
6. Recover the loopback action to original one.

### Test cases #3 - Verify loopback action is saved after reboot(config reload reboot, fast-reboot)
1. Configure the loopback action on the interface, some to drop, some to forward.
2. Config save -y
3. Do reboot randomly(config reload/reboot/warm-reboot/fast-reboot)
4. Verify the loopback action is saved correctly with show cli command.
5. Send traffic to do the validation: 
   - Verify the traffic will be dropped as expected.
   - The RIF drop counter will increase as expected
6. Recover the loopback configuration.

### Test cases #4 Negative test cases
1. Config the loopback action on the port which is not ip interface
2. Check the cli will return expected err msg.
3. Config the loopback action to a value not belong to drop and forward on ip interface
4. Check the cli will return expected err msg.
