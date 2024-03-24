# *__Amplification Attacks__*

There are actually some clever ways to perform a _ping flood_ without using massive amounts of _bandwidth_. An __amplification attack__ uses _spoofing_ and _broadcast addressing_ to amplify a single stream of packets by a _hundred-fold_. First, a target amplification system must be found. This is a network that allows communication to the broadcast address and has a relatively high number of active hosts. Then the attacker sends large _ICMP echo request_ packets to the broadcast address of the amplification network, with a spoofed source address of the victim’s system. The amplifier will broadcast these packets to all the hosts on the amplification network, which will then send corresponding _ICMP echo reply_ packets to the spoofed source address (i.e., to the victim’s machine).

This amplification of traffic allows the attacker to send a relatively small stream of ICMP echo request packets out, while the victim gets swamped with up to a couple hundred times as many ICMP echo reply packets. This attack can be done with both ICMP packets and _UDP echo_ packets. These techniques are known as _smurf_ and _fraggle attacks_, respectively.

<div align="left" width="100%">
<img src="Amplification_Attack.png?raw=true" alt="Amplification Attack" width="100%">
</div>