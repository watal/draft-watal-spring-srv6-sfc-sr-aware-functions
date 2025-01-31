---
title: "SRv6 SFC Architecture with SR-aware Functions"
abbrev: "SRv6 SFC with SR-aware Functions"
docname: draft-watal-spring-srv6-sfc-sr-aware-functions-latest
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

* Comprehensive Management: a centralized controller for SFC, handling SR Policy, link-state, and network metrics.
* Simplicity: no SFC proxies, so that reduces nodes and address resource consumption.

--- middle

# Introduction
Segment Routing over IPv6 (SRv6) {{!RFC8986}} enables packet steering through a set of instructions called a segment list.
Each SR segment endpoint node provides SRv6 Endpoint Behaviors, including Prefix/Adjacency Segments, VPNs, and Binding Segments.

Service Function Chaining (SFC) {{!RFC7665}} can be used in various scenarios (e.g. FW, IPS, IDS, NAT, and DPI).
SFC based on Segment Routing (SR) is defined in {{!I-D.draft-ietf-spring-sr-service-programming}}, which describes some SRv6 Endpoint Behaviors, such as End.AS/AD/AM, are necessary for using SR-unaware functions.

This document describes an architecture for SRv6 SFC with SR-aware functions, which provides comprehensive management of SRv6 network resources and services.

# Terminology

## Terminology Defined in Related RFCs and Internet-Drafts
The following terms are used in this document as defined in the related RFCs and Internet-Drafts:

* SR, SR Domain, Segment ID (SID), SRv6, SR Policy, Prefix Segment, Adjacency Segment, Anycast Segment, Active Segment, and Distributed/Centralized/Hybrid Control Plane defined in {{!RFC8402}}.
* SR Source Node, Transit Node, and SR Segment Endpoint Node defined in {{!RFC8754}}.
* SRv6 Endpoint Behavior defined in {{!RFC8986}}.
* SFC, SFC Proxy, and Service Classification Function defined in {{!RFC7665}}.
* Service Segment, SR-Aware Service, SR-Unaware Service, End.AS, End.AD and End.AM defined in {{!I-D.draft-ietf-spring-sr-service-programming}}.
* Headend, Color, and Endpoint defined in {{!RFC9256}}.
* Quality of Service (QoS), Service Level Agreement (SLA), and Service Level Objective (SLO) defined in {{!RFC9522}}.
* Forwarding Plane, Control Plane, Management Plane, Application Plane defined in {{!RFC7426}}.
* Path Computation Client (PCC), Path Computation Element (PCE), and Traffic Engineering Database (TED) defined in {{!RFC5440}}.
* BGP Flow Specification defined in {{!RFC8955}}

## Newly Defined Terminology
The following terms are used in this document as defined below:

* Service Function Node: an SR segment endpoint node that provides SR-aware functions as service segments.
* SRv6 Controller: controls SRv6 Forwarding Plane, consisting of a PCE and a Classification Rule Controller.
* Classification Rule Controller: applies sets of SR Policy and flows to SR source nodes.
* Service Function Manager: configures network function instances, enables SR-aware functions as service segments, and collects network metrics.

## Requirements Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all capitals, as shown here.

# Design Objectives and Assumptions
## Goals/Objectives
SRv6 SFC Architecture is designed with two main objectives:

* Comprehensive Management: a centralized controller for SFC, handling SR Policy, link-state, and network metrics.
  When providing SRv6 services, meeting SLAs for each customer is required.
  These SLAs consist of one or more SLOs such as availability, latency, and bandwidth.
  In an SRv6 SFC network, service segment provisioning, link-state collection, and SR Policy calculation are required to meet SLOs, respectively.

  {{!RFC8402}} outlines a hybrid control plane that merges a distributed control plane and a centralized control plane.
  In this hybrid control plane, forwarding information like Node/Adjacency SIDs are advertised mutually by distributed SR nodes via IGPs such as ISIS and OSPF, while other information like SR Policies, classification rules, and service segments are provided by a centralized controller and manager.

  Software-Defined Networking (SDN) {{!RFC7426}} provides centralized management of a network by a controller and a manager.
  Centralized management reduces operational costs through abstraction and automation.
  The SDN framework allows users to manage an SR domain without considering the details of a forwarding plane like a topology and node state.
  Operators can use an SRv6 controller to build SR Policies for SFC and QoS, manage the state of network functions, issue service segments automatically, and specify disaster recovery with protection.

* Simplicity: no SFC proxies, so that reduces nodes and address resource consumption.
  Network complexity increases operating costs.
  Generally, using a variety of protocols in a network raises operational costs, including designing, building, monitoring, and troubleshooting.

  Using an SFC proxy may increase forwarding overhead due to additional header manipulations.

## Assumptions
To achieve these objectives, this architecture is based on two main assumptions:

* Straightforward extension of the SRv6 network programming model

  The protocol used in this architecture is compatible with SRv6.
  This streamlines the operation of services like traffic steering, including SFC, redundancy, and local protection.
  Standardized protocols such as BGP, PCEP, IS-IS, OSPF, TI-LFA, and Anycast SID are used in this architecture.

  This architecture is SRv6 compliant, enabling support for SR-unaware functions, although SR-aware functions are expected to meet the objective.

* SDN framework compliance and comprehensive management of SRv6 SFC by controllers

  A controller is used to provide comprehensive management.
  To simplify building and operating, the controller uses standardized protocols and abstracted service interfaces.
  This also provides programmability by controlling policies that meet a user's intent including SFC and quality of service (QoS).

#  Overview of Architecture
Figure 1 illustrates an overview of this architecture.

~~~ drawing
 +----------------------- Application Plane ----------------------+
 |                        User Application                        |
 +-----------------------------------|----------------------------+
                                     |
 +- Control Plane (SRv6 Controller) -v-----+ +- Management Plane -+
 | +--------------+ +--------------------+ | | +----------------+ |
 | |Classification| |        Path        | | | |    Service     | |
 | |     Rule     | |     Computation    | | | |    Function    | |
 | |  Controller  | |    Element (PCE)   | | | |    Manager     | |
 | +------|-------+ +-^-------|--------^-+ | | +----------------+ |
 +--------|-----------|-------|--------|---+ +---------|----------+
          |           |       |        |               |
 +--------|-----------|-------|--------|---------------|---+
 | +------v-----------|-------v-+    +-|---------------v-+ |
 | |                            |    |      Service      | |
 | |       SR Source Node       |----|      Function     | |
 | |                            |    |        Node       | |
 | +----------------------------+    +-------------------+ |
 +------------------- Forwarding Plane --------------------+
~~~
{: #overview title="Overview of SRv6 SFC Architecture with SR-aware Functions"}

This architecture is based on {{!RFC7426}} and consists of forwarding plane, control plane, management plane, and application plane.

* Forwarding Plane: classifies packets and encapsulates SRH, forwards them, and applies SRv6 Endpoint Behavior.
   * Provides SR-aware function using End.AN.
   * Classify flow and apply them to TE application with PBR.
   * Ensures redundancy with anycast.
   * Ensure local protection with Fast Reroute (FRR).
* Control Plane: makes decisions about packet forwarding and provides rules for a forwarding plane.
   * Collects link-state including SRv6 locator, prefix, behavior, and delay.
   * Calculates and provisioning SR Policies.
   * Applies SR Policies to each flow by provisioning flow classification rules.
* Management Plane: deploys and monitors network functions and devices.
   * Setups network functions.
   * Collects metrics of devices, network functions, and SFC services.
* Application Plane: provides APIs for users to use a control and management plane.
   * Provide an interface to operators or customers.
   * Applying intents defined in {{!RFC9315}}, including Operational, Rule, Service, and Flow intents.

Each component communicates using standardized protocols.
These are designed to be loosely coupled and cooperate by using an abstraction layer.

This document suggests handling a control plane by application plane, but a detailed design of an application plane is out of the scope of this document.
This is because application plane components and abstraction layers should be designed based on individual network utilization and operator intent.
In the following sections, details of a forwarding plane, control plane, and management plane are explained.

# Forwarding Plane
A forwarding plane provides SFC through packet classification, SRv6 encapsulation, and forwarding.
In this architecture, all forwarding plane components are located within the SR domain.

~~~ drawing
 +-----------------------------------------------------------------+
 | +-----------+             +----------+             +----------+ |
 | |           | SRv6 Packet | Service  | SRv6 Packet | Service  | |
 | | SR Source |(S2,S1; SL:1)| Function |(S2,S1; SL:1)| Function | |
-->|    Node   |------------>|   Node   |------------>|   Node   |-->
 | |           |             |   (S1)   |             |   (S2)   | |
 | +-----------+             +----------+             +----------+ |
 +--------------------------- SR domain ---------------------------+
~~~
{: #fp title="Forwarding Plane"}

Figure 2 shows an example of SFC with two network functions.
Firstly, the SR source node classifies the flow and encapsulates it with an SRH containing the segment list <S1, S2>.
Next, the service function node (S1) receives the packet and applies a network function associated with an End.AN S1.
Finally, the service function node (S2) receives the packet and also applies a network function associated End.AN S2, thus achieving SFC.

## End.AN-based Service Segment Provisioning
End.AN provides an SR-aware function.

Functions with the same role MAY be assigned as the same service segment within the SR domain.
By using Anycast SIDs, multiple nodes can be grouped as part of the same service segment.

End.AN MAY have optional arguments.
This can provide additional programmability by embedding network function instructions in the segment list.

By using virtualized spaces within routers or on generic servers, network functions can be provided at any node in an SR domain.
This allows for scaling and flexible redundancy of network functions.

### When a Network Function Goes Down
If a network function fails, the associated route MUST be removed immediately.
In the case of anycast configuration, the packet MUST be gracefully rerouted to other nodes.
If no alternative nodes are available, consider either dropping the packet and sending an ICMP Destination Unreachable message, or forwarding it as a pass-through.

### Anycast Segment
The concept of the Anycast Segment is introduced in {{!RFC8402}}.
In the SRv6 SFC, it realizes to provide the same network function segment as the same Anycast Segment.
In such cases, the state between network functions MUST be shared mutually.

### Fast Reroute
The ordering of network functions in an SRv6 SFC is guaranteed by the segment list, even if an FRR occurs,
When an FRR occurs, if the Active segment is an Anycast SID, it MAY be forwarded to another service function node.
In such a case, since state synchronization may not have been completed, the network function MUST have a mechanism to handle rerouted packets, such as buffering to wait for synchronization.

## Service Function Chains
In this architecture, each SFC is represented as an SR Policy.
The purpose or intent of each SR Policy can be identified using attributes such as color or name.

In general, SFC is achieved by using loose source routing.
If both SFC and QoS are desired, they can be achieved by using strict source routing or loose source routing with Flex-Algo SIDs.

## Per-Flow Encapsulation
In an SR source node, which serves as the Service Classification Function, packets are classified on a per-flow basis using PBR and encapsulated with SR Policy.
Therefore, the SR source node MUST be capable of identifying packets using at least a 5-tuple or even more detailed information.

In this architecture, aiming for comprehensive management, the Service Classification Function has an API to communicate with the controller.

# Control Plane
A control plane sets up a forwarding plane by creating SR policies, including SFCs, and applying them to each flow.

~~~ drawing
 +- Control Plane (SRv6 Controller) -------+
 | +--------------+ +--------------------+ |
 | |Classification| |        Path        | |
 | |     Rule     | |     Computation    | |
 | |  Controller  | |    Element (PCE)   | |
 | +------|-------+ +-^-------|--------^-+ |
 +--------|-----------|-------|--------|---+
  Classification link-state SR Policy link-state(Service Segment)
        Rule      (BGP-LS) (PCEP/BGP) (BGP-LS)
  (BGP Flowspec)      |       |        |
 +--------|-----------|-------|--------|-------------------+
 | +------v-----------|-------v-+    +-|-----------------+ |
 | |                            |    |      Service      | |
 | |       SR Source Node       |----|      Function     | |
 | |                            |    |        Node       | |
 | +----------------------------+    +-------------------+ |
 +------------------- Forwarding Plane --------------------+
~~~
{: #cp title="Control Plane"}

The SRv6 Controller consists of the following two components:

* PCE: provides SR Policies that fulfill SFC/QoS requirements from the headend to the tailend and sends them to the SR source node.
* Classification Rule Controller: provides an Encapsulation Policy that corresponds to a specific flow and SR Policy, and sends them to the SR source node.

## Path Computation Element (PCE)
PCE is a controller that provides SR Policy.
As an Active Stateful PCE, it establishes sessions with all PEs in an SR domain and manages SFCs.
SR Policies MUST support both explicit and dynamic paths.
For dynamic path, Constrained Shortest Path First (CSPF) considers not only SFC but also QoS.

It acquires the Traffic Engineering Database (TED) of the SR domain using BGP-LS and deploys SR Policies via PCEP {{!RFC5440}} or BGP SR Policy {{!I-D.draft-ietf-idr-segment-routing-te-policy}}.
The BGP-LS service segment is required to calculate dynamic paths based on the state of service segments and network functions.

## Classification Rule Controller
A Classification Rule Controller determines flows to apply specific SFC.

The classification results are advertised to each SR source node as a set of flow, endpoints, and color with an extended protocol based on BGP Flowspec defined in {{!I-D.draft-ietf-idr-ts-flowspec-srv6-policy}}.

# Management Plane
A management plane configures network function instances, enables SR-aware functions as service segments, monitors resources, and collects network metrics.
The details of each manager are outside the scope of this document, as the southbound interface of the management plane may be different for each service and hardware architecture.

~~~ drawing
 +-------------------- Function Managers ---------------------+
 | +-----------+ +--------------+ +-----------+ +-----------+ |
 | |  Service  | | Virtualized  | |    VNF    | |  Network  | |
 | |  Function | |Infrastructure| |  Manager  | |  Metric   | |
 | |  Manager  | |   Manager    | |           | |  Manager  | |
 | +-----|-----+ +------^-------+ +-----|-----+ +-----^-----+ |
 +-------|--------------|---------------|-------------|-------+
         |              |               |             |
             Management Plane Southbound Interfaces
         |              |               |             |
 +-------|--------------|---------------|-------------|--------+
 | +-----v--------------v---------------v-------------|------+ |
 | |                  Service Function Node                  | |
 | +---------------------------------------------------------+ |
 +------------------------- SR domain -------------------------+
~~~
{: #mp title="Management Plane"}

Figure 4 shows examples of managers that MAY be added to a management plane:

* Service Function Manager: provides an SID for a network service and manages this state.
* VNF Manager: handles deployment and scaling of network functions.
   * VNF Manager keeps links redundant and optimize link utilization.
* VIM: monitors hypervisor resources on service function nodes.
   * In SRv6 SFC, a hypervisor managed by a VIM MAY be located in virtualized spaces within routers or on generic servers.
* Network Metrics Manager: collects metrics for SR Policy calculation and evaluation.
   * Metrics are collected from multiple data sources, including IPFIX, TCP statistics, and SRv6 path tracing {{!I-D.draft-filsfils-spring-path-tracing}}.
   * Metrics can be used for PCE calculation parameters.

## Service Function Manager
Service Function Manager enables and disables service segments of service function nodes.
To manage service segments, it utilizes the extensions provided in a BGP-LS service segment, as outlined in {{!I-D.draft-ietf-idr-bgp-ls-sr-service-segments}} and TODO: draft-watal-idr-bgp-ls-sr-service-segments-enabler, and defines the following parameters:

* Behavior: End.AN
* SID: the SID of End.AN (in IPv6 Address format). Service segments that support slicing are specified here as Flex-Algo SIDs.
* Function Name: type of network function
* Action: enable
* TLV:
    * Specification of the Anycast Group: when deploying multiple Network Functions within the same context, it MUST use the Anycast Group TLV to specify the same anycast group SID.
    * Allows for the specification of unique parameters and context associated with a particular network function.

--- back

# Acknowledgments
{:numbered="false"}
The authors would like to acknowledge the review and inputs from Mitsuru Maruyama, Katsuhiro Sebayashi, Yuma Ito, and Taisei Tanabe.
We partially obtained the research results from NICT's commissioned research No.JPJ012368C03101 and JST's CRONOS No.JPMJCS24N9.
