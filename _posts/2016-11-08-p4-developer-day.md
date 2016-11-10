---
layout: post
title:  "P4 Developer Day"
date:   2016-11-08 15:39:21 -0400
comments: true
---
I recently attended P4 Developer Day at Stanford University. The single-day event was sponsored by a handful of SDN-related companies, some industry/consumer driven, and others derived from academic work like P4 itself. Coming into the day I had only read the [original paper](http://www.sigcomm.org/sites/default/files/ccr/papers/2014/July/0000000-0000004.pdf) _Programming Protocol-Independent Packet Processors_, but I encourage people to check out the [p4lang](http://github.com/p4lang/p4lang) repo and follow the tutorials directory. I apologize for any aliteration that follows, but with a paper title as such, it is inevitable.

<br />

SDN/OpenFlow
===

Let's talk about [OpenFlow](https://www.opennetworking.org/sdn-resources/openflow) quickly. OpenFlow is a standardized communication protocol for software-defined networks (SDNs). It allows network devices (controllers, switches, middleboxes, etc.) to communicate packet processing rules with each other. This gives network administrators control over dataplane behavior, whereas traditional networks conflate control- and data-planes into a single black-box of behavior.

OpenFlow presented the first successful specification of programmable control-plane features, and that is great. You can reprogram routing protocols without reprovisioning your physical infrastructure. But OpenFlow's original specification has grown unwieldy with the continual introduction of support for packet types and switch capabilities.

<br />

P4
===
This is where P4 enters the SDN scene. Rather than updating a spec each year to support increasingly complex packet behavior, necessitating switch updates to support new features, P4 proposes programmer-defined packet parsing functions and match-action table behavior. There is a lot of complexity in how to implement this efficiently, but the high level idea is a natural successor to what OpenFlow started.

Headers and Parsers
---

P4 programs can easily capture what traditional ethernet, ipv4/6, vlan headers, and the rest look like. In the style of a c-struct, the programmer defines the organization of bits in a header, and how to extract information from headers or delve deep into nested headers. Here's what an ethernet header looks like in P4:

```c
header_type ethernet_t {
    fields {
        dstAddr : 48;
        srcAddr : 48;
        ethType : 16;
    }
}
```

so an ethernet header is a 48-bit destination mac followed by a 48-bit source and a 16-bit ethtype. We can define IP and vlan headers similarly, and tell our parser to extract the ethernet frame, and parse the following bits of the packet as an IP header if the appropriate ethtype matches. We can easily unfold nested headers to perform L2/L3 routing as we wish, or perform more complex actions such as tunnel introspection.

<br />

Match-Action Tables
---

For people familiar with OpenFlow, match-action tables are nothing new. P4 supports defining custom actions and matching against packet header or metadata fields. Given a couple of action definitions and assuming we keep around some metadata, we can define a table as follows:

```c
action _drop() {
    drop();
}

action ip_firewall_to_ctrlr() {
    send_to_ctrlr();
}

action ip_pass_through() {
    l3_route();
}

table firwall {
    reads {
        ipv4 : valid;
        ipv4.srcAddr : exact;
    }
    actions {
        _drop;
        ip_firewall_to_ctlr;
        ip_pass_through;
    }
}
```

Of course, you also need to install concrete match values for things such as `ipv4.srcAddr`. But this is the sort of expressiveness and modularity you get in traditional programming environments - P4 is bring that to SDNs.

<br />

Hairyness
===

P4 does come with a handful of rough patches too.

<ul>
<li>Parsing support for things such as variable length header fields is difficult.</li>
<li>If we can program packet header descriptions and table structures, how do we accurately deploy those on highly specialized hardware?</li>
<li>Lacking an operational model, we are left to investing immense effort in testing.</li>
</ul>
<br />
I want to expand a little on deploying to hardware. Prof. [Nate Foster](www.cs.cornell.edu/~jnfoster), from Cornell, spoke about compiler internals, and specifically touched upon the need for API generation for each compilation of a P4 program. This API gets dynamically loaded onto switches, along with the new  match-action tables, so that switches know how to talk about the new packet structures implemented. Without this, switches would not be able to communicate to each other about what sorts of metadata they contain, or how to add or delete rules from their tables. So with P4, we need programmable packet parsing and table definitions, but we need programmable NICs as well.

Automatic API generation can be done a handful of ways. Foster compared an approach of essentially monomorphizing API functions for each match/action pair (giving you strong type guarantees at the cost of a huge number of functions), against a more lenient approach generating generic functions that more or less are able to operate on any table or header spec. With this approach, you gain a more concise API, but lose type-checking measures to protect it because you are essentially operating over a single giant enum. This is just programming languages at work, and there's a rich foundation of work we could apply to make better sense of this.
