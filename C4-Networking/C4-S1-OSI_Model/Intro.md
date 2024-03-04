# *__OSI Model__*

When two computers talk to each other, they need to speak the same language. The structure of this language is described in layers by the __OSI model__. The _OSI model_ provides standards that allow hardware, such as routers and firewalls, to focus on one particular aspect of communication that applies to them and ignore others. The _OSI model_ is broken down into conceptual layers of communication. This way, routing and firewall hardware can focus on passing data at the lower layers, ignoring the higher layers of data encapsulation used by running applications. The seven _OSI layers_ are as follows:

- __Physical layer:__ This layer deals with the physical connection between two points. This is the lowest layer, whose primary role is communicating raw bit streams. This layer is also responsible for activating, maintaining, and deactivating these bit-stream communications.

- __Data-link layer:__ This layer deals with actually transferring data between two points. In contrast with the physical layer, which takes care of sending the raw  bits, this layer provides high-level functions, such as error correction and flow control. This layer also provides procedures for activating, maintaining, and deactivating data-link connections.

- __Network layer:__ This layer works as a middle ground; its primary role is to pass information between the lower and the higher layers. It provides addressing and routing.

- __Transport layer:__ This layer provides transparent transfer of data between systems. By providing reliable data communication, this layer allows the higher layers to never worry about reliability or cost-effectiveness of data transmission.

- __Session layer:__ This layer is responsible for establishing and maintaining connections between network applications.

- __Presentation layer:__ This layer is responsible for presenting the data to applications in a syntax or language they understand. This allows for things like encryption and data compression.

- __Application layer:__ This layer is concerned with keeping track of the requirements of the application.

