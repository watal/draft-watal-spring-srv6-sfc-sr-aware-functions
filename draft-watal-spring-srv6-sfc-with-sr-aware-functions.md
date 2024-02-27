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
* Comprehensive management: centralized controller handles SFC Provisioning and manages LinkState and network metrics.

--- middle

# Introduction
Segment Routing over IPv6 (SRv6) {{!RFC8986}} enables packet steering through a set of instructions called a segment list.
Each SR segment endpoint node provides SRv6 Endpoint Behaviors, including Prefix/Adjacency segments, VPNs, and Binding Segments.

Service Function Chaining (SFC) {{!RFC7665}} can be used in various scenarios (e.g. FW, IPS, IDS, NAT, and DPI).
The SFC based on Segment Routing (SR) is defined in {{!I-D.draft-ietf-spring-sr-service-programming}}, which describes SFC proxies like End.AS/AD/AM are necessary to use SR-unaware functions.

This document describes an architecture for providing SRv6 SFC with SR-aware functions, which provides comprehensive management of an SRv6 network including resources and services but does not define a new protocol.
This architecture satisfies several requirements described in section 3.2.

# Terminology

## Newly Defined Terminology
The following terms are used in this document as defined below:

* SFC Provisioning: deploy service segments associated with network functions, compute SR Policies to satisfy requirements, and deploy to SR Source Nodes.
* SRv6 Service Function Node: an SR segment endpoint node that provides SR-aware functions as service segments.
* Classification Rule Controller: applies sets of SR policy and flow to SR Source Nodes.
* Service Function Controller: applies and manages service segments to SRv6 Service Function Node.
* Service Function Manager: consists of VNF Manager, VIM, and data collector of network metrics.

## Terminology in Related RFCs and Internet-Drafts
The following terms are used in this document as defined in the related RFCs and Internet Drafts:

* Path Computation Client (PCC), Path Computation Element (PCE), and Traffic Engineering Database (TED) defined in {{!RFC5440}}.
* Forwarding Plane (FP), Control Plane (CP), Management Plane (MP), Application Plane (AP), Northbound Interface, Southbound Interface and Service Interface defined in {{!RFC7426}}.
* SFC, SFC Proxy, service classification function, and SFC control plane defined in {{!RFC7665}}.
* Segment Routing (SR), SR Domain, Segment ID (SID), SRv6, SR Policy, Prefix segment, Adjacency segment, Anycast segment, and SR controller defined in {{!RFC8402}}.
* Virtualized Network Function (VNF), VNF Manager, and Virtualized Infrastructure Manager (VIM) defined in {{!RFC8568}}.
* SR source node, transit node, and SR segment endpoint node defined in {{!RFC8754}}.
* SRv6 SID function and SRv6 Endpoint behavior defined in {{!RFC8986}}.
* SRv6 SID function and SRv6 Endpoint behavior defined in {{!RFC8986}}.
* Headend, Color, and Endpoint defined in {{!RFC9256}}.
* egress node, ingress node, metric, measurement methodology, provisioning, Quality of Service (QoS), Service Level Agreement (SLA), and Traffic-engineering system defined in {{!RFC9522}}.
* service segment, SR-aware service, SR-unaware Service, End.AS, End.AD and End.AM defined in {{!I-D.draft-ietf-spring-sr-service-programming}}.

## Requirements Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all capitals, as shown here.

# Design Objectives and Requirements
## Goals/Objectives
The architecture is based on two objectives:

* Simplicity: network complexity increases various costs, including building, operating, and learning costs.
For example, if a network consists of many components, not only do equipment and maintenance costs increase, but also operating costs increase due to increased numbers of points of failure.
Furthermore, a variety of components is also a cause of complexity.
If many types of nodes or protocols are used, the cost of configuration, learning, and monitoring increases.
Hence, this architecture is designed to minimize building blocks and use only SRv6-aware components.

* Comprehensive Management: for SRv6 SFC provisioning, it is important to have a consistent policy for managing service functions, constructing SFC chains, and applying them to each customer.
This requires centralized management of Segment Lists, per-flow steering, network functions, LinkState, and network metrics.
{{!RFC7426}} defines Software-Defined Networking (SDN), which provides consistency and programmability through plane separation and abstraction layer interfaces.
Hence, this architecture is designed to comply with the SDN Framework to provide consistent operation and programmability.

## Requirements
To achieve these objectives, several key requirements are as follows:

o Provide SFC using SR-aware function.

SR-aware function MUST be used to realize simple SFC without proxies.
This minimizes a number of SFC components such as nodes, address resources, and protocols.

To satisfy this requirement, this architecture uses End.AN.

o Straightforward extension of the SRv6 Network Programming model.

The protocol used in this architecture MUST be compatible with SRv6.
This simplifies the operation of services such as traffic steering including SFC, redundancy, and Fast Re-route (FRR).

To satisfy this requirement, this architecture uses standardized SRv6 protocols such as BGP, PCEP, IS-IS, OSPF, TI-LFA, and Anycast SID.

o Comprehensive SRv6 SFC management with controller.

To provide a consistent policy, a controller MUST be used.
To simplify building and operating, the controller MUST use standardized protocols and abstracted service interfaces.
The controller manages not only SR-aware functions but also SR-unaware functions and other SRv6-TE services.
This also provides programmability by controlling policies that satisfy a user's intent including SFC and quality of service (QoS).

To satisfy this requirement, this architecture manages service segments, SFCs, TEs, VPNs, LinkState, and network metrics with a controller.

# Overview of Architecture
Figure 1 and Figure 2 show overviews of SFC with SR-aware and SR-unaware functions, respectively.

~~~ drawing
 +------------------------------------------------+
 |               Application Plane                |
 +------------------------|-----------------------+
                          | Service Interface
 +- SRv6 SFC Controllers -v-----------------------+
 | +--------------+ +-------------+ +-----------+ | +-----------+
 | |Classification| |    Path     | | Service   | | |  Service  |
 | |     Rule     | | Computation | | Function  | | |  Funtion  |
 | |  Controller  | |Element (PCE)| |Controller | | |  Managers |
 | +------|-------+ +-^---------|-+ +-----|-----+ | +-----|-----+
 +--------|-----------|---------|---------|-------+       | Management
          |           |         |         |               | Plane
         Control Plane Southbound Interfaces              | Southbound
          |           |         |         |               | Interface
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
 | +-------------------------------+ +---------------------------+ |
 | |     SRv6 SR Source Node /     | |                           | |
 | |    Service Classification     |-|        SFC Proxy          | |
 | |           Function            | |                           | |
 | +-------------------------------+ +----^---------------|------+ |
 +----------- SRv6 domain ----------------|---------------|--------+
                                          |               |
                                     +----|---------------v------+
                                     |        SR-unaware         |
                                     |         Function          |
                                     |                           |
                                     +---------------------------+
~~~
{: #srv6-sfc-with-sr-unaware-functions title="SRv6 SFC with SR-unaware functions"}

This architecture is based on SDN {{!RFC7426}} separating the Forwarding Plane (FP), Control Plane (CP), Management Plane (MP), and Application Plane (AP).
Each plane has the following roles:

* FP: responsible for providing an SR-aware network, classifying services, and applying SFC for each flow.
   * Provide SRv6-aware function using End.AN.
   * Flow classification and TE application with PBR.
   * Redundancy and protection with Anycast and Fast Reroute.
* CP: responsible for controlling Service Segment, calculating SR Policy including SFC, and providing classification rules for each flow.
   * Collecting LinkState including SRv6 locator, prefix, behavior, and delay.
   * Calculating and provisioning SR Policies.
   * Applying SR Policies to each flow by provisioning flow classification rule.
   * Provisioning Service Segments to SR-aware functions.
* MP: responsible for deploying SR-aware functions, managing resources, and collecting network metrics.
   * Monitoring and deploying network functions.
   * Managing hypervisor resources.
   * Collecting metrics about devices, network functions, and SFC service.
* AP: responsible for providing application interfaces to specify user intent, topology visualization, and notification.
   * Provide an interface to operators or customers.
   * Applying intents defined in {{!RFC9315}}, including Operational, Rule, Service, and Flow intents.

Each component communicates using standardized protocols as described in section 3.2.
These components are designed to be loosely coupled and cooperate by using an abstraction layer.

This document suggests handling CP by AP, but a detailed design of AP is out of the scope of this document.
This is because AP components and abstraction layers should be designed based on individual network utilization and operator intent.
In the following sections, details of FP, CP, and MP are explained.

# Forwarding Plane
The forwarding plane is responsible for applying SFC through packet classification, SRv6 encapsulation, and forwarding.

~~~ drawing
 +----------------------------------------------------------------+
 | +--------------+             +--------+             +--------+ |
 | |   SRv6 SR    | SRv6 Packet |  SRv6  | SRv6 Packet |  SRv6  | |
 | | Source Node  |(S2,S1; SL:1)|Service |(S2,S1; SL:1)|Service | |
-->|  / Service   |------------>|Function|------------>|Function|--->
 | |Classification|             |  Node  |             |  Node  | |
 | |   Function   |             |  (S1)  |             |  (S2)  | |
 | +--------------+             +--------+             +--------+ |
 +------------------------- SRv6 domain --------------------------+
~~~
{: #fp title="Forwarding Plane"}

Figure 3 shows an example of SFC with two network service functions.
Firstly, the SRv6 SR source node classifies the flow and encapsulates it with an SRH containing the segment list <S1, S2>.
Next, the SRv6 Service Function Node (S1) receives the packet and applies a network function associated End.AN.
Finally, the SRv6 Service Function Node (S2) receives the packet and also applies a network function associated End.AN, thus achieving SFC.

SRv6 proxy を用いない転送を実現している。

ビルディングブロックが減るため、トラシューが楽だよ
また、これは遅延を減らす可能性もあるよ

また、SRv6 domain で完結した転送を実現するため、後述する C-Plane で

## End.AN-based Service Segment Provisioning
By using End.AN at an SR segment endpoint node, End.AN can be realized for providing service segments natively in SRv6.
Simultaneously, this enables the provisioning of network service functions at any location based on customer demand within the SR domain.

Functions with the same role MAY be assigned as the same service segment within the SR domain.
By using Anycast-SIDs, multiple nodes can be grouped as part of the same service segment.

End.AN MAY have optional arguments that can be passed as parameters to bound network service functions.

### Anycast Segment
The concept of the Anycast segment is introduced in {{!RFC8402}}.
It is permissible to configure the same network service function segment as the same Anycast segment.
In such cases, the state between network service functions MUST be shared mutually.

### When a Network Service Function Goes Down
If a network service function experiences a failure, the associated route MUST be promptly removed.
In the case of Anycast configuration, it MUST be gracefully rerouted to other nodes.
Additionally, if no alternative nodes are available, consider either dropping the packet and sending an ICMP Destination Unreachable message or forwarding it as a pass-through.

### Fast Re-route
Because SFCs are structured as a Segment List, the order of application is guaranteed even in the event of FRR for functions.
In such cases, if Anycast segments are used, it is permissible to take a detour to a more optimal node.

## Service Function Chains
In this architecture, each SFC is represented as an SRv6 Policy {{!RFC9256}}.
The purpose or intent of each SRv6 Policy can be identified using attributes such as color or name.

## Per-Flow Encapsulation
In the SRv6 SR source node, which serves as the Service Classifier, packets are classified on a per-flow basis using PBR and encapsulated with SRv6 Policy.
Therefore, the SRv6 SR source node MUST be capable of identifying packets using at least a 5-tuple or even more detailed information.

# Control Plane
CP is

~~~ drawing
 +------------- SRv6 SFC Controllers -------------+
 | +--------------+ +-------------+ +-----------+ |
 | |Classification| |    Path     | | Service   | |
 | |     Rule     | | Computation | | Function  | |
 | |  Controller  | |Element (PCE)| |Controller | |
 | +------|-------+ +-^---------|-+ +-----|-----+ |
 +--------|-----------|---------|---------|-------+
    Classification LinkState SR Policy  Enable/Disable
         Rule      (BGP-LS) (PCEP/BGP) a Service Segment
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

The SRv6 Controllers consists of the following three components:

* Service Function Controller: this component is responsible for defining the state and SID of network service functions on an SRv6 Service Function Node and managing the service segment.
* SRv6 Policy Manager: this component generates SR Policies that fulfill SFC/QoS requirements from the headend to the tailend and sends them to the SRv6 SR source node.
* Classification Rule Controller: this component generates an Encapsulation Policy that corresponds to a specific flow and SR Policy, and sends them to the SRv6 SR source node.

コントロールプレーンのいいところを述べる

## Service Function Controller
The Service Function Controller is responsible for enabling and disabling service segments of SRv6 Service Function Nodes.
To manage service segments, it utilizes the extensions provided in BGP-LS service segment, as outlined in {{!I-D.draft-ietf-idr-bgp-ls-sr-service-segments}} and {{!I-D.draft-watal-idr-bgp-ls-sr-service-segments-enabler}}, and defines the following parameters:

* Behavior: End.AN
* SID: the SID of End.AN (in IPv6 Address format). Service segments that support slicing are specified here as Flex-Algo SIDs.
* Function Name: type of network service function
* Action: enable
* TLV:
    * Specification of the Anycast Segment Group: when deploying multiple Network Functions within the same context, it MUST use the Anycast Group TLV to specify the same anycast segment group SID.
    * Allows for the specification of unique parameters and context associated with a particular network service function.

これを用いることで何をするかを述べる

## Path Computation Element (PCE)
なぜ必要かを述べる

SRv6 Active Stateful PCE
It acquires the Traffic Engineering Database (TED) of the SRv6 domain using BGP-LS and deploys SR Policies via PCEP {{!RFC5440}} or BGP SR Policy {{!I-D.draft-ietf-idr-segment-routing-te-policy}}.

The SR Policy can utilize CSPF to satisfy various requirements, including SFC and QoS.
SR Policies can be defined on a per-flow or per-TE basis, providing flexibility.

## Classification Rule Controller
なぜ必要かを述べる

For communication with each node, an extended protocol based on BGP Flow Spec is used for SR Policy.
SR Policy specification consists of three components: endpoint, color, and policy name.

The set of endpoints and color is transmitted as described in {{!I-D.draft-ietf-idr-ts-flowspec-srv6-policy}}.

# Management Plane
MP is responsible for configuring instances, monitoring resources, and maintaining services.
MP Southbound Interfaces are specific to individual services and hardware architectures, therefore, details on each manager are outside the scope of this document.

~~~ drawing
 +--------------- SRv6 SFC Managers ---------------+
 | +--------------+ +--------------+ +-----------+ |
 | | Virtualized  | |     VNF      | |  Network  | |
 | |Infrastructure| |   Manager    | |  Metric   | |
 | |   Manager    | |              | |  Manager  | |
 | +------^-------+ +------^-------+ +-----^-----+ |
 +--------|----------------|---------------|-------+
          |                |               |
        Management Plane Southbound Interfaces
          |                |               |
 +--------|----------------|---------------|-------+
 | +------|----------------v---------------|-----+ |
 | |                 SRv6 Service                | |
 | |                   Function                  | |
 | |                     Node                    | |
 | +---------------------------------------------+ |
 +------------------- SRv6 domain -----------------+
~~~
{: #mp title="Management Plane"}

Figure 5 shows examples of managers that MAY be added to MP:

* VNF Manager: handles deployment and scaling of network functions.
   * This manager MAY consider redundancy and link utilization optimization.
   * This manager complies with {{!RFC8568}}.
* Virtualized Infrastructure Manager (VIM): monitors hypervisor resources on SRv6 Service Function Node.
   * This manager complies with {{!RFC8568}}.
* Network Metrics Manager: collects metrics for SRv6 policy calculation and evaluation.
   * Metrics are collected from multiple data sources, including SRv6 path traces, IPFIX, and TCP statistics.
   * Metrics can be used as inputs for controllers described in this document.

# Security Considerations
In this architecture, network functions are globally accessible via IPv6, since the network functions are SRv6 service segments.
If a network function has a security vulnerability, this node could be attacked, and other nodes in the SRv6 domain could also be lateral movement attacks.
Therefore, by default, the information of each service segment MUST NOT be leaked outside of a domain, network operators MUST use filtering to drop packets from unauthorized sources to service segments.

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
In the Interop Tokyo 2023 ShowNet's backbone SRv6 network, they had to decapsulate packets to conduct Network Address Translation.
If you use SR-aware NAT, you don't have to decapsulate the packets when traversing the NAT function.
This contributes to achieving a simpler network design.

# Intent-based SFC management
{{!RFC9315}} defines intent as "operational guidance and information about the goals, purposes, and service instances that the network is to serve.
The architecture for providing SRv6 SFC with SR-aware functions is based on the SDN Framework {{!RFC7426}} and includes an Application Plane.

TODO

# Acknowledgments
{:numbered="false"}
The authors would like to acknowledge the review and inputs from Mitsuru Maruyama, Katsuhiro Sebayashi, Yuma Ito, and Taisei Tanabe.
We partially obtained the research results from NICT's commissioned research No. JPJ012368C03101.
