# Ryu
This folder contains implementations of the All-Path family protocols based on the Ryu SDN framework: https://osrg.github.io/ryu/

### All-Path basic locking
The file *allpath_switch_13.py* is based on the Ryu's SimpleSwitch application for OpenFlow 1.3 (simple_switch_13.py), dated 02/2016. It extends it so that it avoids loops in the network. To do so, the name of the application has been changed to AllPathSwitch13 and two parts of code have been added to the original source:

First, the All-Path locking to avoid loops:
```
	# all-path lock
	if src in self.mac_to_port[dpid] and in_port != self.mac_to_port[dpid][src]:
	    self.logger.info("---discarded! (%s, %s)", in_port, self.mac_to_port[dpid][src])
	    return
```
(It simply checks whether the address has already been learnt or not. If learnt and the associated port is different, it discards the frame as a late copy <- the ones that cause loops)

Second, if BOOTP (or any other switching protocol) is active, we need to deactivate it:
```
	# no BOOTP
	if eth.ethertype == ether_types.ETH_TYPE_IP and dst == mac.BROADCAST_STR:
	    ip = pkt.get_protocols(ipv4.ipv4)[0]
	    if ip.proto == in_proto.IPPROTO_UDP:
		self.logger.info("---discarded! (src = %s, dst = %s)", src, dst)
		return
```
(Notice this is just a quick fix, to have the less modifications to simple_switch_13.py as possible. But this could be done in a more elegant way)

This implementation extends Ryu's SimpleSwitch13 to avoid loops. Additionally, a path repair mechanism could be implemented (not done yet).
