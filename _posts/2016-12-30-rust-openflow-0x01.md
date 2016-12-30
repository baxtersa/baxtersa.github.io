---
layout: post
title: "OpenFlow 1.0 Protocol in Rust"
date: 2016-12-30 12:21:53 -0400
comments: true
---
I've published a [crate](https://crates.io/crates/rust_ofp) (my first!) implementing a large portion of the OpenFlow 1.0 protocol in Rust. I am surprised I was not able to find any packages either natively implementing software-defined networking (SDN) capabilities in Rust, or providing Rust bindings to existing protocol libraries written in other languages. `rust_ofp` takes the first step into this empty space of Rust packages, providing a Rust-native implementation of the [OpenFlow 1.0 specification](http://archive.openflow.org/documents/openflow-spec-v1.0.0.pdf), and offering traits that abstract the core functionality an OpenFlow controller should provide.

<br />

Why Rust?
===
I'll preface this by stating that I am a programming language nerd, and I currently work on the [frenetic-lang](www.frenetic-lang.org) project developing tools and abstractions for NetKAT (a compiled network programming language based on sold mathematical foundations). I'm _not_ a network administrator, and am more interested in linguistic abstractions that guarantee network properties and grow expressiveness than I am in _being_ a network administrator. This is mostly a comparison of what Rust offers that could benefit the frenetic project. So, take my perspective with that in mind.

At the lowest level, implementing an OpenFlow protocol naturally fits the low-level (zero-cost) abstractions of Rust. Having control over memory layout makes implementing a wire protocol easy. With

```rust
#[repr(packed)]
struct OfpActionVlanPcp(u8, [u8; 3]);
```

you can acheive C-style struct layout without the compiler inserting padding for field alignment, and without jumping through the `CStruct` hoops of frenetic's OCaml implementation to avoid the same problem:

```ocaml
[%%cstruct
type ofp_action_vlan_pcp = {
  vlan_pcp: uint8_t;
  pad: uint8_t [@len 3];
} [@@big_endian]]
```

OCaml CStruct's automatic code generation for setters/getters of cstruct fields is nice, but choosing constrained integer representations for enums in Rust hasn't been bad either. Manipulating a `&mut [u8]` feels more comfortable to me when serializing/deserializing bytes.

Furthermore, Rust's performance can be much easier to reason about. This may derive from my days as a C++ developer, but Rust's explicit use of pointers and references make it clear that I am not copying around payloads and message abstractions, but reusing the same data where possible.

For me, the difference between functors in OCaml and traits in Rust doesn't amount to much. I've found it somewhat easier to implement a common interface amongst different message types using traits, but I'm accustomed to functors and modules, so my design in Rust borrows heavily from the functional paradigm and existing frenetic implementation.

The main selling point for a Rust frenetic platform is true parallelism. OCaml notoriously doesn't have good support for parallelism. Long-running computations (compiling a new NetKAT policy on a dynamic configuration update) either run the risk of hogging controller responsiveness, or require non-idiomatic OCaml using unix threads in combination with pervasive use of JaneStreet's Async library. Rust can isolate each controller-switch connection to individual threads, and Rust's concurrency model gives you great guarantees about the correctness of parallel code. I'm looking forward to experimenting with Rust concurrency to see how an SDN controller can reap these benefits.

<br />

rust_ofp
===
`rust_ofp` is a library implementing the OpenFlow 1.0 protocol. What does it look like? At the highest level of abstraction, it provides traits for implementing common OpenFlow structures for different protocol versions:

 - `trait OfpHeader` for OpenFlow message headers.
 - `trait OfpMessage` for byte-level operations on OpenFlow message types.
 - `trait OfpController` for OpenFlow controller operations, like connection handshakes and communicating with switches over `TcpStream`s.

I have so far implemented what I consider the majority of useful functionality for an OpenFlow 1.0 SDN in the `openflow0x01` module. This module implements the `OfpMessage` trait for OpenFlow 1.0 messages, and the accompanying binary crate `rust_ofp_controller` gives a small example using the library to install rules when a switch connects to the controller.

A controller implementor can add flows with the `FlowMod` message type:

```rust
pub enum Message {
...
FlowMod(FlowMod),
PacketIn(PacktIn),
...
}
```
...

```rust
pub struct FlowMod {
  pub command: FlowModCmd,
  pub pattern: Pattern,
  pub actions: Vec<Action>,
  ...
}
```
`FlowMod`s take a struct specifying a `FlowModCmd` that determines the type of modification to make to a switch's flowtable (add, modify, delete), along with a pattern and list of actions to perform on pattern matches. An empty action list is interpreted as the `DROP` action.

`PacketIn` message types (i.e. packets that arrive at the controller) can be handled to perform behaviors like MAC learning, dynamically installing rules on switches using network discovery.

The library design is intended to allow other OpenFlow specifications to implement the same traits, enabling a pluggable back-end for a single SDN compiler to target multiple protocol versions. I hope to implement OpenFlow 1.3 (specifically for multi-table support), but will otherwise be focusing on front-end abstractions to SDN specifications once the protocol is in working order.

<br />

Documentation
===
Travis CI integration in the [github repo](https://github.com/rust_ofp) automatically uploads source documentation generated by `cargo doc`.

 - [`rust_ofp`](https://baxtersa.github.io/rust_ofp/docs)
 - [`rust_ofp_controller`](https://baxtersa.github.io/rust_ofp/docs/rust_ofp_controller)

Please give a shot at implementing your own SDN controller using `rust_ofp`! If there's demand for it, I'll cover setting up and testing a controller in the mininet network simulation environment in a future post. Get in touch if you want to discuss anything further!
