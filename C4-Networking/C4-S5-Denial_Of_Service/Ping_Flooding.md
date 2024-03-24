# *__Ping Flooding__*

Flooding DoS attacks don’t try to necessarily crash a service or resource, but instead try to overload it so it can’t respond. Similar attacks can tie up other resources, such as CPU cycles and system processes, but a _flooding attack_ specifically tries to tie up a network resource.

The simplest form of flooding is just a __ping flood__. The goal is to use up the victim’s bandwidth so that legitimate traffic can’t get through. The attacker sends many large ping packets to the victim, which eat away at the bandwidth of the victim’s network connection.

There’s nothing really clever about this attack—it’s just a battle of bandwidth. An attacker with greater bandwidth than a victim can send more data than the victim can receive and therefore deny other legitimate traffic from getting to the victim.