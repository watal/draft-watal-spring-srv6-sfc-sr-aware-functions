---
title: "SRv6 SFC Architecture with SR-aware Functions"
abbrev: "SRv6 SFC with SR-aware Functions"
docname: draft-watal-spring-srv6-sfc-with-sr-aware-functions-latest
category: info

ipr: trust200902
area: Routing
workgroup: SPRING
keyword:
  - SRv6
  - SFC
stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: W. Mishima
    name: Wataru Mishima
    organization: NTT Communications
    email: w.mishima@ntt.com
    country: Japan
 -
    ins: Y. Fukagawa
    name: Yuta Fukagawa
    organization: NTT Communications
    email: y.fukagawa@ntt.com
    country: Japan

normative:

informative:

--- abstract

This document describes the architecture of SRv6 Service Function Chaining (SFC) with SR-aware functions.
This architecture provides the following benefits:

* Simplicity: no SFC Proxies which reduces components such as nodes and address resources.
* Comprehensive management: centralized controller handles SFC Provisioning and manages link-state and network metrics.

--- middle

# Introduction
Segment Routing over IPv6 (SRv6) {{!RFC8986}} enables packet steering through a set of instructions called a segment list.
Each SR segment endpoint node provides SRv6 Endpoint Behaviors, including Prefix/Adjacency segments, VPNs, and Binding Segments.

Service Function Chaining (SFC) {{!RFC7665}} can be used in various scenarios (e.g. FW, IPS, IDS, NAT, and DPI).
The SFC based on Segment Routing (SR) is defined in {{!I-D.draft-ietf-spring-sr-service-programming}}, which describes SFC proxies like End.AS/AD/AM are necessary to use SR-unaware functions.

This document describes the architecture of SRv6 SFC with SR-aware functions, which provides comprehensive management of an SRv6 network including resources and services.
The document does not define any new protocol but defines an architecture for providing SFC with SR-aware functions that satisfy several requirements described in section 3.1.

# Terminology
## Related RFCs and Internet-Drafts
This document uses terminologies defined in the following documents:

* {{!RFC5440}} describes the Path Computation Element Communication Protocol (PCEP) and defines the following terms: Path Computation Client (PCC), Path Computation Element (PCE), and Traffic Engineering Database (TED).
* {{!RFC7426}} describes the Software-Defined Networking layer architecture and defines the following terms: Forwarding Plane (FP), Control Plane (CP), Management Plane (MP), and Application Plane (AP).
* {{!RFC7665}} describes the SFC architecture and defines the following terms: SFC, SFC Proxy, service classification function, and SFC control plane.
* {{!RFC8402}} describes the Segment Routing architecture and defines the following terms: Segment Routing (SR), SR Domain, Segment ID (SID), SRv6, SR Policy, Prefix segment, Adjacency segment, Anycast segment, and SR controller.
* {{!RFC8568}} describes open research challenges for network virtualization including ETSI NFV Framework and defines following terms: Virtualized Network Function (VNF), VNF Manager and Virtualized Infrastructure Manager (VIM).
* {{!RFC8754}} describes the encoding of IPv6 segments in the SRH and defines the following terms: SR source node, transit node, and SR segment endpoint node.
* {{!RFC8986}} describes the main SRv6 behaviors and defines the following terms: SRv6 SID function and SRv6 Endpoint behavior.
* {{!RFC9256}} describes the SR Policy architecture and defines the following terms: Headend, Color, and Endpoint.
* {{!RFC9522}} describes the principle of internet traffic engineering and defines the following terms: egress node, ingress node, metric, measurement methodology, provisioning, Quality of Service (QoS), Service Level Agreement (SLA), and Traffic-engineering system.
* {{!I-D.draft-ietf-spring-sr-service-programming}} describes the SFC based on SR and defines the following terms: service segment, SR-aware service, SR-unaware Service, End.AS, End.AD and End.AM.

## Newly Defined Terminology
The following terms are used in this document as defined below:

* SFC Provisioning: to provide SFC as a service, deploy service segments to network functions, build SFC to satisfy a policy, and deploy to SR Source Node.
* SRv6 Service Function Node: an SR segment endpoint node that provides SR-aware functions as service segments.
* Service Function Controller: a CP component which
* Service Function Controller: a MP component which
* Classification Rule Controller: a CP component which

## Requirements Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all capitals, as shown here.

# Design Objectives and Requirements
## Objectives
This architecture has the following objectives:

* Simplicity
   * no SFC Proxies which reduces components such as nodes and address resources.
   * TE, redundancy, and Fast Re-route (FRR) are achieved by using SRv6, which does not require any additional protocols.
* Comprehensive Management
   * centralized controller handles SFC Provisioning and manages link-state and network metrics.
   * control using standardized protocols.
   * manages not only SR-aware functions but also SR-unaware functions and other SRv6-TE services.

## Requirements
To achieve these objectives, several key requirements as follows:

1. Service segment for SR-aware function: using End.AN, provide SFC without SR Proxies.
2. Centralized policy management: including service segments, SFCs, TEs, VPNs, LinkState, network metrics .
3. Provide programmability: provide flexibility
4. Straightforward extension: using SRv6 standard protocols such as BGP, PCEP, without any changes

TODO: add a description

# Overview of SRv6 Native SFC Architecture
In Figure 1 and Figure 2, the Overview of SFC with SR-aware and SR-unaware functions, respectively.

~~~ drawing
 +------------------------------------------------+
 |               Application Plane                |
 +------------------------|-----------------------+
                          | Service Interface
 +--- SRv6 Controllers ---v-----------------------+
 | +--------------+ +-------------+ +-----------+ | +-----------+
 | |Classification| |    Path     | | Service   | | |  Service  |
 | |     Rule     | | Computation | | Function  | | |  Funtion  |
 | |  Controller  | |Element (PCE)| |Controller | | |  Manager  |
 | +------|-------+ +-^---------|-+ +-----|-----+ | +-----|-----+
 +--------|-----------|---------|---------|-------+       |
          |           |         |         | CP            | MP
          |           |         |         | Southbound    | Southbound
          |           |         |         | Interface     | Interface
 +--------|-----------|---------|---------|---------------|-------+
 | +------v-----------|---------v-+ +-----v---------------v-----+ |
 | |     SRv6 SR Source Node /    | |       SRv6 Service        | |
 | |    Service Classification    |-|         Function          | |
 | |           Function           | |           Node            | |
 | +------------------------------+ +---------------------------+ |
 +-------------------------- SRv6 domain -------------------------+
~~~
{: #srv6-sfc-with-sr-aware-functions title="SRv6 SFC with SR-aware functions"}

~~~ drawing
 +-----------------------------------------------------------------+
 | +-------------------------------+ +------------+ +------------+ |
 | |     SRv6 SR Source Node /     | |            | |            | |
 | |    Service Classification     |-| SFC Proxy  |-| SFC Proxy  | |
 | |           Function            | |            | |            | |
 | +-------------------------------+ +---|-----^--+ +---|-----^--+ |
 +------------- SRv6 domain -------------|-----|--------|-----|----+
                                         |     |        |     |
                                     +---v-----|--+ +---v-----|--+
                                     | SR-unaware | | SR-unaware |
                                     |  Function  | |  Function  |
                                     |            | |            |
                                     +------------+ +------------+
~~~
{: #srv6-sfc-with-sr-unaware-functions title="SRv6 SFC with SR-unaware functions"}

This architecture based on Software-Defined Networking (SDN) {{!RFC7426}} separating the Forwarding Plane (FP), Control Plane (CP), Management Plane (MP), and Application Plane (AP).

Each plane has the following roles:

* FP:
* CP:
* MP:
* AP:

Each component communicates using standardized protocols as described in 3.1 Requirements.
These components are designed to be loosely coupled and cooperate by using abstraction layers.

As defined in {{!RFC9315}}, Intent types include Operational, Rule, Service, Flow, and so on.
This document suggests the handling of CP with AP, but the detailed design of the AP and abstraction layer is out of scope of this document.

In the following sections, details of FP, CP, and MP are explained.

# Forwarding Plane
The FP is designed as follows:

It minimizes nodes and reduces addresses, hostnames, and other resources.

{{!RFC7665}} outlines a procedure in which each packet is classified by the service classification function, then forwarded to the Service Function Forwarder, and subsequently delivered to a specific network service function.
In the SRv6 native SFC architecture, the SRv6 SR source node classifies the flow and forwards it to a specific SRv6 Service Function Node by specifying a Segment List that represents a particular SFC.

~~~ drawing
 +----------------------------------------------------------------+
 | +--------------+             +--------+             +--------+ |
 | |   SRv6 SR    | SRv6 Packet |  SRv6  | SRv6 Packet |  SRv6  | |
 | | Source Node  |(S2,S1; SL:1)|Service |(S2,S1; SL:1)|Service | |
-->|  / Service   |------------>|Function|------------>|Function|--->
 | |Classification|             |  Node  |             |  Node  | |
 | |   function   |             |  (S1)  |             |  (S2)  | |
 | +--------------+             +--------+             +--------+ |
 +------------------------- SRv6 domain --------------------------+
~~~
{: #fp title="Forwarding Plane"}

Figure 3 shows an example of SFC with two network service functions.
Firstly, the SRv6 SR source node classifies the flow and encapsulates it with an SRH containing the segment list <S1, S2>.
Next, the SRv6 Service Function Node for S1 receives the packet and applies End.AN.
Finally, the SRv6 Service Function Node for S2 receives the packet and also applies End.AN, thus achieving SFC.

Deploying multiple instances of the same network service function in an SRv6 domain enables the implementation of SFC as a multipath.
In such scenarios, stateful network service functions like FW or NAT MUST establish state-sharing mechanisms among SRv6 Service Function Nodes.
Additionally, service segments that provide stateless network service functions can achieve geographically efficient delivery by utilizing Anycast-SIDs.

## End.AN-based Service Segment Provisioning
By utilizing End.AN at an SR segment endpoint node, End.AN can be realized for providing service segments natively in SRv6. Simultaneously, this enables the provisioning of network service functions at any location based on customer demand within the SR domain.

Functions with the same role MAY be assigned as the same service segment within the SR domain.
By using Anycast-SIDs, multiple nodes can be grouped as part of the same service segment.

End.AN MAY have optional arguments that can be passed as parameters to bound network service functions.

### Anycast Segment
The concept of the Anycast segment is introduced in {{!RFC8402}}. It is permissible to configure the same network service function segment as the same Anycast segment.
In such cases, the state between network service functions MUST be shared mutually.

### When a Network Service Function Goes Down
If a network service function experiences a failure, the associated route MUST be promptly removed. In the case of Anycast configuration, it MUST be gracefully rerouted to other nodes.
Additionally, if no alternative nodes are available, consider either dropping the packet and sending an ICMP Destination Unreachable message or forwarding it as a pass-through.

### Fast Reroute
Because SFCs are structured as a Segment List, the order of application is guaranteed even in the event of Fast ReRoute for functions.
In such cases, if Anycast segments are used, it is permissible to take a detour to a more optimal node.

## SRv6 Policy
In SRv6 Native SFC, each SFC is represented as an SRv6 Policy {{!RFC9256}}.
The purpose or intent of each SRv6 Policy can be identified using attributes such as color or name.

## Per-Flow Encapsulation
In the SRv6 SR source node, which serves as the Service Classifier, packets are classified on a per-flow basis using PBR and encapsulated with SRv6 Policy.
Therefore, the SRv6 SR source node MUST be capable of identifying packets using at least a 5-tuple or even more detailed information.

# Control Plane
The CP is designed as follows:

~~~ drawing
 +--------------- SRv6 Controllers ---------------+
 | +--------------+ +-------------+ +-----------+ |
 | |Classification| |    Path     | | Service   | |
 | |     Rule     | | Computation | | Function  | |
 | |  Controller  | |Element (PCE)| |Controller | |
 | +------|-------+ +-^---------|-+ +-----|-----+ |
 +--------|-----------|---------|---------|-------+
        Encap     LinkState SR Policy  Enable/Disable
       Policy     (BGP-LS) (PCEP/BGP) a Service Segment
    (BGP Flowspec)    |         |     (End.AN SID:S1)
 +--------|-----------|---------|---------|-----------------------+
 | +------v-----------|---------v-+ +-----v---------------------+ |
 | |     SRv6 SR Source Node /    | |       SRv6 Service        | |
 | |    Service Classification    |-|         Function          | |
 | |           Function           | |           Node            | |
 | +------------------------------+ +---------------------------+ |
 +-------------------------- SRv6 domain -------------------------+
~~~
{: #cp title="Control Plane"}

The SRv6 Native SFC Controller consists of the following three components:

* Service Function Controller: this component is responsible for defining the state and SID of network service functions on an SRv6 Service Function Node and managing the service segment.
* SRv6 Policy Manager: this component generates SR Policies that fulfill SFC/QoS requirements from the headend to the tailend and sends them to the SRv6 SR source node.
* Classification Rule Controller: this component generates an Encapsulation Policy that corresponds to a specific flow and SR Policy, and sends them to the SRv6 SR source node.

## Service Function Controller
The Service Function Controller is responsible for enabling and disabling service segments of SRv6 Service Function Nodes.
To manage service segments, it utilizes the extensions provided in BGP-LS service segment, as outlined in {{!I-D.draft-ietf-idr-bgp-ls-sr-service-segments}} and {{!I-D.draft-watal-idr-bgp-ls-srv6-sfc-enabler}}, and defines the following parameters:

* Behavior: End.AN
* SID: the SID of End.AN (in IPv6 Address format). Service segments that support slicing are specified here as Flex-Algo SIDs.
* Function Name: type of network service function
* Action: enable
* TLV:
    * Specification of the Anycast Segment Group: when deploying multiple Network Functions within the same context, it MUST use the Anycast Group TLV to specify the same anycast segment group SID.
    * Allows for the specification of unique parameters and context associated with a particular network service function.

## Path Computation Element (PCE)
SRv6 Active Stateful PCE
It acquires the Traffic Engineering Database (TED) of the SRv6 domain using BGP-LS and deploys SR Policies via PCEP {{!RFC5440}} or BGP SR Policy {{!I-D.draft-ietf-idr-segment-routing-te-policy}}.

The SR Policy can utilize CSPF to satisfy various requirements, including SFC and QoS.
Moreover, SR Policies can be defined on a per-flow or per-TE basis, providing flexibility.

## Classification Rule Controller
For communication with each node, an extended protocol based on BGP Flow Spec is used for SR Policy.
SR Policy specification consists of three components: endpoint, color, and policy name.

The set of endpoints and color is transmitted as described in {{!I-D.draft-ietf-idr-ts-flowspec-srv6-policy}}.

# Management Plane

~~~ drawing
 +--------------- SRv6 Controllers ---------------+
 | +--------------+ +-------------+ +-----------+ | +-----------+
 | |Classification| |    Path     | | Service   | | |  Service  |
 | |     Rule     | | Computation | | Function  | | |  Funtion  |
 | |  Controller  | |Element (PCE)| |Controller | | |  Manager  |
 | +------|-------+ +-^---------|-+ +-----|-----+ | +-----|-----+
 +--------|-----------|---------|---------|-------+       |
          |           |         |         | CP            | MP
          |           |         |         | Southbound    | Southbound
          |           |         |         | Interfaces    | Interfaces
 +--------|-----------|---------|---------|---------------|-------+
 | +------v-----------|---------v-+ +-----v---------------v-----+ |
 | |     SRv6 SR Source Node /    | |       SRv6 Service        | |
 | |    Service Classification    |-|         Function          | |
 | |           Function           | |           Node            | |
 | +------------------------------+ +---------------------------+ |
 +-------------------------- SRv6 domain -------------------------+
~~~
{: #mp title="Management Plane"}

{{!RFC8568}}

TODO: add a description

Additional managers that can be added to the SRv6 Native SFC Controller MAY include:

* Service Function Manager: handles the deployment of network functions to SRv6 Service Function Nodes.
* Virtualized Infrastructure Manager: managing aspects like memory usage on the hypervisor that provides network functions and monitors the available resources.
* Metric Manager: collect metrics to evaluate SRv6 Policy some collection methods described in {{!RFC9232}}. e.g. SRv6 Path Tracing, IPFIX, TCP statistics.

The metrics collected by these other managers can be used as inputs for managers described in this document.
Details on each specific manager are outside the scope of this document.

# Security Considerations
In this architecture, network functions are globally accessible via IPv6, since the network functions are SRv6 service segments.
If a network function has a security vulnerability, this node or other devices on the IPv6 network could be attacked.
Therefore, by default, the information of each service segment MUST NOT be leaked to outside of domain, network operators MUST use filtering to drop packets from unauthorized sources to service segments.

The security requirements and mechanisms described in {{!RFC8402}}, {{!RFC8754}}, and {{!RFC8986}} are also applicable to this document.

# IANA Considerations
This document has no IANA actions.

--- back

# Highly Reliable Firewall Service Using SRv6 End.AN
If you implement a firewall as an SR-aware function at an SRv6 End.AN node, you can forward packets using anycast SID and also achieve TI-LFA Fast Reroute {{!I-D.draft-ietf-rtgwg-segment-routing-ti-lfa}}.
This makes clustering firewalls easier as well.

# Flexible and Low-latency Remote Production Service
In the context of video remote production, you can perform video processing within an SRv6 network by combining multiple network functions (SFC).
If you have to distribute multiple connections from several sources, you can also use multicast packets in the SRv6 network.

# SR-aware NAT
In the Interop Tokyo 2023 shownet's backbone SRv6 network, they had to decapsulate packets to conduct Network Address Translation.
If you use SR-aware NAT, you don't have to decap the packets when traversing the NAT function.
This contributes to achieving a simpler network architecture(design?)

# Intent-based SFC management
TODO

# User-defined functions
TODO

# Acknowledgments
{:numbered="false"}
The authors would like to acknowledge the review and inputs from Mitsuru Maruyama, Katsuhiro Sebayashi, Yuma Ito, and Taisei Tanabe.
We partially obtained the research results from NICT's commissioned research No. JPJ012368C03101.
