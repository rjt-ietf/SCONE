---
title: "Client Conformance Signal for SCONE"
category: info

docname: draft-rjt-scone-conformance-signal-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Web and Internet Transport"
workgroup: "Standard Communication with Network Elements"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: "Standard Communication with Network Elements"
  type: "Working Group"
  mail: "scone@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/scone"
  github: "rjt-ietf/SCONE"
  latest: "https://rjt-ietf.github.io/SCONE/draft-rjt-scone-conformance-signal.html"

author:
 -
    fullname: "Renjie Tang"
    organization: Google
    email: "rjt.ietf@gmail.com"

normative:

informative:


--- abstract

This document proposes conformance signals to be sent by QUIC clients to indicate whether they are able to adapt to the bitrate indicated by the SCONE signal, so that communication service providers MAY stop policing.


--- middle

# Introduction

As outlined in the charter, the primary objective of SCONE is to facilitate the communication of throughput suggestions between traffic throttling network elements and QUIC endpoints. One of the main motivations is the desire to disable traffic shapers and policers when possible. However, the ability for communication service providers (CSPs) to unidirectionally send throughput advice signals to QUIC endpoints does not provide the CSPs with information on whether the QUIC endpoints are conforming to the suggested throughput. As a result, the CSP has no assurance to disable throttling.

A [paper](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/45411.pdf) was published on the prevalence and harmfulness of throughput throttling. From an ISP's perspective, it requires additional machine resources to run traffic shapers and policers, and dropped packets result in waste of network bandwidth. From a QUIC server's perspective, retransmissions incur extra costs to server resources. From a QUIC client's perspective, packet loss harms the user's quality of experience. Although communicated throughput advice SHOULD prevent packets from being dropped by traffic policers most of the time, short-duration bursts are common within certain types of network connections, causing the problems mentioned above. 

In addition to determining the format and delivery method for throughput advice, the working group should also establish the conditions under which CSPs SHOULD deactivate their traffic shapers and transition into trust-and-verify mode, where average throughputs are sampled to make sure the content and application providers (CAPs) are behaving as expected.


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Proposals

The following proposals assume the throughput advice is transmitted from network elements to QUIC clients in the format of a [TRAIN](https://datatracker.ietf.org/doc/draft-thomson-scone-train-protocol/) packet. As the technical design of SCONE evolves, the proposals in this draft MAY evolve as well. If the reasoning in this draft is well received, conformance signals MAY also be integrated into the SCONE technical solution proposal drafts.


## Implicit Signal

The traffic throttling network element SHOULD default marks the 4-tuple flow as conformant when its TRAIN packet is received by the QUIC client. The network element SHOULD not disable traffic shapers until it confirms the QUIC client has acked the SCONE signal. Since the network element lacks visibility into QUIC packets containing ACK frames, it MAY only deduce the QUIC client's receipt of the signal by observing the cessation of TRAIN packet retransmissions by the QUIC server. In the case where the network element gives an unrealistically low throughput advice and the QUIC client decides to not conform, the client SHOULD not ack the TRAIN packets. The SCONE protocol SHOULD also specify a limit on the number of SCONE packet retransmissions. When the retransmission limit is reached, the QUIC server MUST not retransmit any more TRAIN packets, and the network element SHOULD consider the current flow ineligible for SCONE and keeps its throttling device on.


# Explicit Signal

The QUIC client SHOULD signal conformance by echoing back the TRAIN packet. Upon receiving the TRAIN packet, if the decision is to conform, the QUIC client SHOULD send back the same TRAIN packet along with its ACKs to the QUIC server. When the client-initated TRAIN packet reaches the network element, it SHOULD be dropped and throttling devices SHOULD be disabled. In the case where the QUIC client decides to not conform, it SHOULD NOT echo the TRAIN packet back. Since the network element never receives the conformance signal, it keeps its throttling devices on.


# Security Considerations

The transmission of the conformance signal must employ the same security protection mechanism utilized for the original SCONE packets.


# IANA Considerations

This document has no IANA actions.


--- back
