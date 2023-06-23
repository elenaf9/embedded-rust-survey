# SmolTCP
- != `smol::net::`

<https://m-labs.hk/software/smoltcp/>
- designed for bare-metal, real-time systems
- no heap allocation

<https://docs.rs/smoltcp/latest/smoltcp/>

- "toolbox of networking primitives"
layered structure; every layer exposed:
- Socket layer: 
  - usual primitives 
  - buffering, packet construction & validation, state machines for stateful sockets
  - interface-agnostic, application mus use socket together with network interface
  - raw, ICMP, TCP & UCP sockets; but not conform with Berkeley socket API because that needs heap allocation
- Interface layer:
  - control messages, physical addressing & neighboar discovery
  - routes packets to and from sockets
  - ethernet interface provided
- Physical Layer:
  - raw sockets & TAP interface provided
  - also 2 middleware interface:
    - tracer device: prints human-readable repesentation of packets
    - fault injector device: randomly introduce erros into transmitted and recieved packets
  - handles interaction with a platform-specific network device
- Wire Layers: "make illegal states irrepresentable"
  - allow construction wire layer objects that can be parsed from or emitted to a lower layer
  - representation layer:
    - reduce state space of raw packets
    - shed nonsensical packets (i.e. invalid checksum, meaningless options, unsupported features, ...)
  - Packet layer
    - more structured way to work with packets
  
## Implementation

<https://github.com/smoltcp-rs/smoltcp/tree/master>

- Socket implementation: Sans-IO



