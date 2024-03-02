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

This document describes the architecture of Segment Routing over IPv6 (SRv6) Service Function Chaining (SFC) with SR-aware functions.
This architecture provides the following benefits:

* Comprehensive management: a centralized controller for SFC, handling SR Policy, link-state, and network metrics.
* Simplicity: no SFC Proxies, so that reduces nodes and address resource consumption.

--- middle

# Introduction
Segment Routing over IPv6 (SRv6) {{!RFC8986}} enables packet steering through a set of instructions called a segment list.
Each SR segment endpoint node provides SRv6 Endpoint Behaviors, including Prefix/Adjacency segments, VPNs, and Binding Segments.

Service Function Chaining (SFC) {{!RFC7665}} can be used in various scenarios (e.g. FW, IPS, IDS, NAT, and DPI).
The SFC based on Segment Routing (SR) is defined in {{!I-D.draft-ietf-spring-sr-service-programming}}, which describes SFC proxies like End.AS/AD/AM are necessary to use SR-unaware functions.

This document describes an architecture for SRv6 SFC with SR-aware functions, which provides comprehensive management of an SRv6 network including resources and services.

# Terminology

## Terminology Defined in Related RFCs and Internet-Drafts
The following terms are used in this document as defined in the related RFCs and Internet-Drafts:

* SR, SR Domain, Segment ID (SID), SRv6, SR Policy, Prefix segment, Adjacency segment, Anycast segment, and Active segment defined in {{!RFC8402}}.
* SR source node, transit node, and SR segment endpoint node defined in {{!RFC8754}}.
* SRv6 SID function and SRv6 Endpoint behavior defined in {{!RFC8986}}.
* SFC, SFC Proxy, and service classification function defined in {{!RFC7665}}.
* service segment, SR-aware service, SR-unaware Service, End.AS, End.AD and End.AM defined in {{!I-D.draft-ietf-spring-sr-service-programming}}.
* Headend, Color, and Endpoint defined in {{!RFC9256}}.
* egress node, ingress node, metric, Quality of Service (QoS), Service Level Agreement (SLA), and Service Level Objective (SLO) defined in {{!RFC9522}}.
* Forwarding Plane (FP), Control Plane (CP), Management Plane (MP), Application Plane (AP), Northbound Interface, Southbound Interface and Service Interface defined in {{!RFC7426}}.
* Path Computation Client (PCC), Path Computation Element (PCE), and Traffic Engineering Database (TED) defined in {{!RFC5440}}.

## Newly Defined Terminology
The following terms are used in this document as defined below:

* SRv6 Service Function Node: an SR segment endpoint node that provides SR-aware functions as service segments.
* Classification Rule Controller: applies sets of SR policy and flows to SR Source Nodes.
* Service Function Controller: applies service segments to SRv6 Service Function Nodes.
* SRv6 SFC Controllers: provide comprehensive management of SRv6 SFC, consisting of a Service Function Controller, a PCE, and a Classification Rule Controller.
* SRv6 SFC Managers: manage SRv6 SFC infrastructure, consisting of a Virtualized Network Function (VNF) Manager, a Virtualized Infrastructure Manager (VIM), and a data collector of network metrics.

## Requirements Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all capitals, as shown here.

# Design Objectives and Requirements
## Goals/Objectives
SRv6 SFC Architecture is designed with two main objectives:

* Comprehensive management: a centralized controller for SFC, handling SR Policy, link-state, and network metrics.
  When providing SRv6 services, meeting SLAs for each customer is required.
  These SLAs consist of one or more SLOs such as availability, latency, and bandwidth.
  In an SRv6 SFC network, service segment provisioning, link state collection, and SR policy calculation are required to meet SLOs, respectively.

  {{!RFC8402}} outlines a hybrid CP that merges distributed and centralized CPs.
  In this hybrid CP, basic segments like Node/Adjacency are distributed via IGP protocols such as ISIS and OSPF, while service-specific states like SR policies and service segments are provided by a centralized controller.

  As an approach to achieving centralized control, {{!RFC7426}} defines Software-Defined Networking (SDN).
  Centralized management of SRv6 SFC components reduces operational costs through abstraction and automation.
  Additionally, programmability can be provided by using SRv6 SFC Controllers and abstracted APs.
  Operators can build SFCs, apply them to specific flows, set SLOs as an intent, and determine fallback policies via controllers' API.

  For these reasons, this architecture is designed to provide comprehensive management of SRv6 SFC.

* Simplicity: no SFC Proxies, so that reduces nodes and address resource consumption.
  Network complexity increases operating costs.
  Generally, using a variety of protocols in a network raises operational costs, including design, building, monitoring, and troubleshooting.

  A complex FP can be a cause of increasing latency.
  In architectures using SFC proxy, forwarding overhead may increase due to additional header manipulations compared to those without proxies.

  SRv6 has various functions such as VPN, QoS, redundancy, and disaster recovery.
  By using SR-aware functions, all forwarding instructions, including instructions to each network function, can be expressed as a simple set and applied to each flow.

  For these reasons, this architecture is designed to minimize FP components and use SRv6-aware network functions.

## Requirements
To achieve these objectives, several key requirements are as follows:

* Provide SFC using an SR-aware function

  An SR-aware function MUST be used to achieve simple SFC without proxies.
  This minimizes the number of nodes, address resources, and protocols.
  This architecture uses End.AN.

* Straightforward extension of the SRv6 Network Programming model

  The protocol used in this architecture MUST be compatible with SRv6.
  This simplifies the operation of services such as traffic steering including SFC, redundancy, and Fast Reroute (FRR).
  This architecture uses standardized SRv6 protocols such as BGP, PCEP, IS-IS, OSPF, TI-LFA, and Anycast SID.

* SDN Framework compliance and comprehensive management of SRv6 SFC by controllers

  A controller MUST be used to provide a consistent policy.
  To simplify building and operating, the controller MUST use standardized protocols and abstracted service interfaces.
  The controller manages not only SR-aware functions but also SR-unaware functions and other SRv6-TE services.
  This also provides programmability by controlling policies that meet a user's intent including SFC and quality of service (QoS).
  This architecture uses controllers to manage service segments, SFCs, TEs, VPNs, link-state, and network metrics.

# Overview of Architecture
Figures 1 and 2 show overviews of SFC with SR-aware and SR-unaware functions, respectively.

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
 +--------------------------- SR domain --------------------------+
~~~
{: #srv6-sfc-with-sr-aware-functions title="SRv6 SFC with SR-aware functions"}

~~~ drawing
 +-----------------------------------------------------------------+
 | +-------------------------------+ +---------------------------+ |
 | |     SRv6 SR Source Node /     | |                           | |
 | |    Service Classification     |-|        SFC Proxy          | |
 | |           Function            | |                           | |
 | +-------------------------------+ +----^---------------|------+ |
 +----------- SR domain ------------------|---------------|--------+
                                          |               |
                                     +----|---------------v------+
                                     |        SR-unaware         |
                                     |         Function          |
                                     |                           |
                                     +---------------------------+
~~~
{: #srv6-sfc-with-sr-unaware-functions title="SRv6 SFC with SR-unaware functions"}

As illustrated in Figure 1, this architecture realizes SFC without proxies.
This architecture is based on SDN {{!RFC7426}} separating the Forwarding Plane (FP), Control Plane (CP), Management Plane (MP), and Application Plane (AP).
Each plane has the following roles:

* FP: responsible for providing SR-aware functions, classifying services, and applying SFCs for each flow.
   * Provide SRv6-aware function using End.AN.
   * Flow classification and TE application with PBR.
   * Redundancy and protection with Anycast and FRR.
* CP: responsible for controlling Service Segment, calculating SR Policy including SFC, and providing classification rules for each flow.
   * Collecting link-state including SRv6 locator, prefix, behavior, and delay.
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

Each component communicates using standardized protocols.
These are designed to be loosely coupled and cooperate by using an abstraction layer.

This document suggests handling CP by AP, but a detailed design of AP is out of the scope of this document.
This is because AP components and abstraction layers should be designed based on individual network utilization and operator intent.
In the following sections, details of FP, CP, and MP are explained.

# Forwarding Plane
FP is responsible for providing SFC through packet classification, SRv6 encapsulation, and forwarding.
In this architecture, all FP components are located within the SR domain.

~~~ drawing
 +----------------------------------------------------------------+
 | +--------------+             +--------+             +--------+ |
 | |   SRv6 SR    | SRv6 Packet |  SRv6  | SRv6 Packet |  SRv6  | |
 | | Source Node  |(S2,S1; SL:1)|Service |(S2,S1; SL:1)|Service | |
-->|  / Service   |------------>|Function|------------>|Function|-->
 | |Classification|             |  Node  |             |  Node  | |
 | |   Function   |             |  (S1)  |             |  (S2)  | |
 | +--------------+             +--------+             +--------+ |
 +-------------------------- SR domain ---------------------------+
~~~
{: #fp title="Forwarding Plane"}

Figure 3 shows an example of SFC with two network functions.
Firstly, the SRv6 SR source node classifies the flow and encapsulates it with an SRH containing the segment list <S1, S2>.
Next, the SRv6 Service Function Node (S1) receives the packet and applies a network function associated with an End.AN S1.
Finally, the SRv6 Service Function Node (S2) receives the packet and also applies a network function associated End.AN S2, thus achieving SFC.

## End.AN-based Service Segment Provisioning
End.AN provides an SRv6-aware network function.

Functions with the same role MAY be assigned as the same service segment within the SR domain.
By using Anycast-SIDs, multiple nodes can be grouped as part of the same service segment.

End.AN MAY have optional arguments.
This can provide additional programmability by embedding network function instructions in the segment list.

### When a Network Function Goes Down
If a network function experiences a failure, the associated route MUST be promptly removed.
In the case of Anycast configuration, it MUST be gracefully rerouted to other nodes.
Additionally, if no alternative nodes are available, consider either dropping the packet and sending an ICMP Destination Unreachable message or forwarding it as a pass-through.

### Anycast Segment
The concept of the Anycast segment is introduced in {{!RFC8402}}.
In the SRv6 SFC, it realizes to provide the same network function segment as the same Anycast segment.
In such cases, the state between network functions MUST be shared mutually.

### Fast Reroute
The ordering of network functions in an SRv6 SFC is guaranteed by the segment list, even if an FRR occurs,
When an FRR occurs, if the Active segment is an Anycast SID, it MAY be forwarded to another SRv6 Service Function Node.
In such a case, since state synchronization may not have been completed, the network function MUST have a mechanism to handle rerouted packets, such as buffering to wait for synchronization.

## Service Function Chains
In this architecture, each SFC is represented as an SRv6 Policy {{!RFC9256}}.
The purpose or intent of each SRv6 Policy can be identified using attributes such as color or name.

In general, SFC is achieved by using loose source routing.
If both SFC and QoS are desired, they can be achieved by using strict source routing or loose source routing with Flex-Algo SIDs.

## Per-Flow Encapsulation
In the SRv6 SR source node, which serves as the Service Classifier, packets are classified on a per-flow basis using PBR and encapsulated with SRv6 Policy.
Therefore, the SRv6 SR source node MUST be capable of identifying packets using at least a 5-tuple or even more detailed information.

In this architecture, aiming for comprehensive management, the service classifier has an API to communicate with the controller.

# Control Plane
CP is responsible for enabling comprehensive management of SRv6 SFC.
It enables SR-aware functions as service segments and specifies SR Policies including SFC for each flow.
CP has a Northbound API to receive user requests and a Southbound API to manipulate FP.

~~~ drawing
 +------------- SRv6 SFC Controllers -------------+
 | +--------------+ +-------------+ +-----------+ |
 | |Classification| |    Path     | | Service   | |
 | |     Rule     | | Computation | | Function  | |
 | |  Controller  | |Element (PCE)| |Controller | |
 | +------|-------+ +-^---------|-+ +-----|-----+ |
 +--------|-----------|---------|---------|-------+
   Classification link-state SR Policy  Enable/Disable
        Rule       (BGP-LS) (PCEP/BGP) a Service Segment
   (BGP Flowspec)     |         |     (End.AN SID:S1)
 +--------|-----------|---------|---------|-----------------------+
 | +------v-----------|---------v-+ +-----v---------------------+ |
 | |     SRv6 SR Source Node /    | |       SRv6 Service        | |
 | |    Service Classification    |-|         Function          | |
 | |           Function           | |           Node            | |
 | +------------------------------+ +---------------------------+ |
 +--------------------------- SR domain --------------------------+
~~~
{: #cp title="Control Plane"}

The SRv6 SFC Controllers consist of the following three components:

* Service Function Controller: provides an SID for a network service and manages this state.
* PCE: provides SR Policies that fulfill SFC/QoS requirements from the headend to the tailend and sends them to the SRv6 SR source node.
* Classification Rule Controller: provides an Encapsulation Policy that corresponds to a specific flow and SR Policy, and sends them to the SRv6 SR source node.

## Service Function Controller
Service Function Controller is responsible for enabling and disabling service segments of SRv6 Service Function Nodes.
To manage service segments, it utilizes the extensions provided in a BGP-LS service segment, as outlined in {{!I-D.draft-ietf-idr-bgp-ls-sr-service-segments}} and {{!I-D.draft-watal-idr-bgp-ls-sr-service-segments-enabler}}, and defines the following parameters:

* Behavior: End.AN
* SID: the SID of End.AN (in IPv6 Address format). Service segments that support slicing are specified here as Flex-Algo SIDs.
* Function Name: type of network function
* Action: enable
* TLV:
    * Specification of the Anycast Segment Group: when deploying multiple Network Functions within the same context, it MUST use the Anycast Group TLV to specify the same anycast segment group SID.
    * Allows for the specification of unique parameters and context associated with a particular network function.

XXX: Service Function Controller を使う利点を説明する

## Path Computation Element (PCE)
PCE is a controller that provides SR Policy.
As an Active Stateful PCE, it establishes sessions with all PEs in an SR domain and manages SFCs.
SR Policies MUST support both explicit and dynamic paths.
For dynamic path, CSPF MUST consider not only SFC but also QoS.

It acquires the Traffic Engineering Database (TED) of the SR domain using BGP-LS and deploys SR Policies via PCEP {{!RFC5440}} or BGP SR Policy {{!I-D.draft-ietf-idr-segment-routing-te-policy}}.

The SR Policy can utilize CSPF to meet various requirements, including SFC and QoS.
SR Policies can be defined on a per-flow or per-TE basis, providing flexibility.
The BGP-LS service segment is needed to calculate the dynamic path considering the service segment and the state of the network function.

## Classification Rule Controller

A Classification Rule Controller specifies flows to apply specific SFC.

For communication with each node, an extended protocol based on BGP Flow Spec is used for SR Policy.
SR Policy specification consists of three components: endpoint, color, and policy name.

The set of endpoints and color is transmitted as described in {{!I-D.draft-ietf-idr-ts-flowspec-srv6-policy}}.

# Management Plane
MP is responsible for configuring network function instances, monitoring resources, and collecting network metrics.
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
 +-------------------- SR domain ------------------+
~~~
{: #mp title="Management Plane"}

Figure 5 shows examples of managers that MAY be added to MP:

* VNF Manager: handles deployment and scaling of network functions.
   * This manager MAY consider redundancy and link utilization optimization.
* Virtualized Infrastructure Manager (VIM): monitors hypervisor resources on SRv6 Service Function Node.
   * In SRv6 SFC, a hypervisor managed by a VIM MAY be located in virtualized spaces within routers or on generic servers.
* Network Metrics Manager: collects metrics for SRv6 policy calculation and evaluation.
   * Metrics are collected from multiple data sources, including SRv6 path traces, IPFIX, and TCP statistics.
   * Metrics can be used as inputs for controllers described in this document.

# Security Considerations
In this architecture, network functions are globally accessible via IPv6, since the network functions are SRv6 service segments.
If a network function has a security vulnerability, this node could be attacked, and other nodes in the SR domain could also be lateral movement attacks.
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

XXX: 文章追加

# Acknowledgments
{:numbered="false"}
The authors would like to acknowledge the review and inputs from Mitsuru Maruyama, Katsuhiro Sebayashi, Yuma Ito, and Taisei Tanabe.
We partially obtained the research results from NICT's commissioned research No. JPJ012368C03101.
