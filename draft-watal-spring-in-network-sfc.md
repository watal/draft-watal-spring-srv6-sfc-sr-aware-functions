---
title: "SRv6 In-Network SFC Architecture"
abbrev: "SRv6 In-Network SFC"
docname: draft-watal-spring-in-network-sfc-latest
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
    organization: NTT Communications Corporation
    email: watal@wide.ad.jp
    country: Japan
 -
    ins: Y. Fukagawa
    name: Yuta Fukagawa
    organization: NTT Communications Corporation
    email: skyline@fkgw.org
    country: Japan

normative:

informative:


--- abstract

Segment Routing over IPv6 (SRv6) {{!RFC8986}} enables packet steering through a set of instructions called the segment list by attaching an SR header to IPv6 packets. Each SRv6 Segment Endpoint Node can provide various SRv6 Endpoint Behaviors, such as Node/Adjacency Segments, VPNs, and Service Function Chaining (SFC) {{!RFC7665}}. This allows network operators to achieve advanced programmability for packets using SR Policies.

"End.AN" {{!I-D.draft-ietf-spring-sr-service-programming}} is an SRv6 behavior that provides SR-aware network service functions. By utilizing this behavior, network service functions can be placed at any position within the SR domain, enabling "In-network SFC" along the shortest path. Placing network service functions at any router allows service provisioning at locations based on demand and ensures services are delivered via the best path, optimizing factors such as minimal latency and bandwidth.

Achieving SFC at any position requires the following control actions:

* Activating network service functions at the SR Segment Endpoint (Enabling End.AN).
* Creating service function chains at the SR Source Node (Adding SR Policy).
* Classifying the target flow at the SR source node (Adding an encapsulation policy).

This document outlines the methodologies for achieving In-network SFC using SR-aware functions, covering operational aspects of the SRv6 data plane and management methods for network service functions, SR Policies, and Encapsulation Policies in the control plane.


--- middle

# Introduction

SFC enhances network functionality by providing various use cases, including security services such as FW, IPS/IDS, NAT services, and at the L7 level, features like DPI and remote video production through packet payload editing.

In the current SRv6 architecture, SR Proxy {{!I-D.draft-ietf-spring-sr-service-programming}} is used for SFC, but it faces two key issues:

* Geographic Constraints: It is incapable of deploying network service functions in desired or specific locations.
* Decreased Forwarding Efficiency: The service function chain cannot be established along the shortest path, resulting in reduced forwarding efficiency.

End.AN is an SRv6 Endpoint behavior targeting at an SR-aware network service function, and enabling a node to process SRv6 natively and apply network service functions. This allows for the provision of network service functions at geographically optimal locations, achieving SFC on the best path. It also enables the utilization of available resources and the possibility for routers themselves to provide network service functions.

This document describes a framework for In-network SFC using SRv6-based methods, covering both the data plane and control plane. The generic SFC architecture is covered in {{!RFC7665}}, while the SFC architecture based on Segment Routing is covered in {{!I-D.draft-li-spring-sr-sfc-control-plane-framework}}, and therefore outside the scope of this document.

# Terminology
This document leverages the terminology proposed in {{!RFC8402}}, {{!RFC8660}}, {{!RFC8754}}, {{!RFC8986}} and {{!RFC9256}}. It also introduces the following new terms.

* SRv6 Service Function Node: An SRv6 Segment Endpoint Node that provides SRv6 native network service functions as a service segment.
* In-network SFC: A technology that achieves SFC at any point of SRv6 domain by deploying SR-aware network service functions on any SRv6 Service Function Node.
* In-network SFC Controller: A controller that simultaneously manages SRv6 Service Function Nodes and SRv6 SR Source Nodes, overseeing the entire SFC within the SRv6 domain.

# Overview of SRv6 In-Network SFC Architecture
In the following sections, the SRv6 In-Network SFC Architecture design is detailed. It begins with the definition of design principles and is followed by the explanation of components and key technologies in both the data plane and control plane.

## SRv6 In-Network SFC Architecture
{{!RFC7665}} outlines a procedure in which each packet is classified by the Service Classification Function, then forwarded to the Service Function Forwarder, and subsequently delivered to a specific network service function.
In the SRv6 native SFC architecture, the SRv6 SR Source Node classifies the flow and forwards it to a specific SRv6 Service Function Node by specifying a Segment List that represents a particular Service Function Chain.

~~~ drawing
   +------------------------------------------------------------------------------+
   |  +--------------+ SRv6 Encapsulated +--------+ SRv6 Encapsulated +--------+  |
   |  |   SRv6 SR    |  Packet (End.AN)  |  SRv6  |  Packet (End.AN)  |  SRv6  |  |
   |  | Source Node  |  (S2, S1; SL:1)   |Service |  (S2, S1; SL:1)   |Service |  |
 ---->|  / Service   |------------------>|Function|------------------>|Function|---->
   |  |Classification|                   |  Node  |                   |  Node  |  |
   |  |   function   |                   |  (S1)  |                   |  (S2)  |  |
   |  +--------------+                   +--------+                   +--------+  |
   +-------------------------------- SRv6 domain ---------------------------------+
~~~
{: #in-network-sfc-data-plane title="In-network SFC (Data Plane)"}

Figure 1 shows an example of SFC with two network service functions.
Firstly, the SRv6 SR Source Node classifies the flow and encapsulates it with an SRH containing the segment list <s1, s2>.
Next, the SRv6 Service Function Node for s1 receives the packet and applies End.AN.
Finally, the SRv6 Service Function Node for s2 receives the packet and also applies End.AN, thus achieving SFC.

Deploying multiple instances of the same network service function in an SRv6 domain enables the implementation of Service Function Chaining as a multipath.
In such scenarios, stateful network service functions like FW or NAT must establish state-sharing mechanisms among SRv6 Service Function Nodes.
Additionally, Service Segments that provide stateless network service functions can achieve geographically efficient delivery by utilizing anycast SIDs.

## Architecture Principles
In-network SFC is based on several key architectural principles:

1. In-network Processing: Forwarding packets along the shortest path.
2. Deployment of Network Service Functions to Any Nodes: Deploying functions to any nodes based on user demand.
3. Integration with Traffic Engineering (TE): Simultaneously representing SFC policy and QoS policy with SR Policy.
4. Per-Flow Identification: To apply SFC Policies per flow, classifying at the SRv6 SR Source Node based on the 5-tuple or even finer granularity.
5. Centralized Management: Managing the entire SRv6 domain by the controller, including aggregating network service functions on the network and per-flow encapsulation policies.
6. Ease of Deployment: Employs existing protocols commonly used in controllers, such as BGP and PCEP.
7. Coexistence with Existing Services: Operating while maintaining core network services like slicing and VPNs.

# Data Plane
The Data Plane is designed as follows to satisfy Architecture Principles 1, 3, and 4:

* End.AN-based Service Segment Provisioning: To achieve in-network processing with SRv6, the data plane utilizes End.AN to handle SRv6-aware functions.
* SRv6 Policy: Achieving SFC and QoS requirements through the Segment List.
* Per-Flow Encapsulation Policy: Applying per-flow requirements through the Encapsulation Policy using PBR.

## End.AN-based Service Segment Provisioning

By utilizing End.AN at an SRv6 Segment Endpoint Node, End.AN can be realized for provide Service Segments natively in SRv6. Simultaneously, this enables the provisioning of network service functions at any location based on customer demand within the SR domain.

Functions with the same role MAY be assigned as the same Service Segment within the SR domain.
By using Anycast SIDs, multiple nodes can be grouped as part of the same Service Segment.

End.AN MAY have optional arguments that can be passed as parameters to bound network service functions.

### End.AN Pseudocode
The "Endpoint with SR-Aware Service" behavior ("End.AN" for short) is an SRv6 behavior that enables the native provision of network service functions to packets.

When N receives a packet whose IPv6 DA is S and S is a local End.AN SID, N does the following:

~~~ drawing
S01. When an SRH is processed {
S02.   If (Segments Left == 0) {
S03.      Proceed to process the next header in the packet.
S04.   }
S05.   If (IPv6 Hop Limit <= 1) {
S06.      Send an ICMP Time Exceeded message to the Source Address
          with Code 0 (Hop limit exceeded in transit),
          interrupt packet processing, and discard the packet.
S07.   }
S08.   max_LE = (Hdr Ext Len / 2) - 1
S09.   If ((Last Entry > max_LE) or
           (Segments Left > Last Entry+1)) {
S10.      Send an ICMP Parameter Problem to the Source Address
             with Code 0 (Erroneous header field encountered)
             and Pointer set to the Segments Left field,
             interrupt packet processing, and discard the packet.
S11.   }
S12.   Apply the network service function associated with the LocalSID
S13.   Decrement IPv6 Hop Limit by 1
S14.   Decrement Segments Left by 1
S15.   Update IPv6 DA with Segment List[Segments Left]
S16.   Submit the packet to the egress IPv6 FIB lookup for
          transmission to the new destination
S17. }
~~~
{: #end-an-pseudocode title="SID processing for SR-aware function (native)"}

### Anycast Segment
The concept of the Anycast Segment is introduced in {{!RFC8402}}. It is permissible to configure the same network service function segment as the same Anycast segment.
In such cases, the state between network service functions SHOULD be shared mutually.

### When a Network Service Function Goes Down
If a network service function experiences a failure, the associated route should be promptly removed. In the case of Anycast configuration, it should be gracefully rerouted to other nodes.
Additionally, if no alternative nodes are available, consider either dropping the packet and sending an ICMP Destination Unreachable message or forwarding it as a pass-through.

### Fast ReRoute
Because Service Function Chains are structured as a Segment List, the order of application is guaranteed even in the event of Fast ReRoute for functions.
In such cases, if Anycast Segments are used, it is permissible to take a detour to a more optimal node.

## SRv6 Policy
In SRv6 In-network SFC, each Service Function Chain is represented as an SRv6 Policy {{!RFC9256}}.
The purpose or intent of each SRv6 Policy can be identified using attributes such as color or name.

## Per-Flow Encapsulation Policy
In the SRv6 SR Source Node, which serves as the Service Classifier, packets are classified on a per-flow basis using PBR and encapsulated with SRv6 Policy.
Therefore, the SRv6 SR Source Node should be capable of identifying packets using at least a 5-tuple or even more detailed information.

# Control Plane
The Control Plane is designed as follows to satisfy Architecture Principles 2, 3, 4, 5, 6, and 7:

* Network Service Functions: Enable/Disable network service functions on any node within the SRv6 domain.
* SRv6 Policy: Calculate constrained paths that achieve both SFC and QoS requirements.
* Per-flow Encapsulation Policy: Classify flows and issue encapsulation policies to achieve per-flow SFC.
* SDN Approach: Centralized management of network service functions, encapsulation policies, and SR Policies by the controller.
* Generic Protocol: Utilizing standardized protocols such as MP-BGP and PCEP, with minimal extensions.
* Integration with Current Network Contexts: Policy identification methods that coexist with existing network contexts, including SR Policy Colors associated with slices, VPNs, and more.

~~~ drawing
   +-------------------------------------- In-network SFC Controller ---------------------------------------+
   |  +-------------+        +-------------------+            +------------------------------------------+  |
   |  |Encapsulation|        |     SR Policy     |            |                                          |  |
   |  |   Policy    |        |      Manager      |            |         Service Function Manager         |  |
   |  |   Manager   |        |       (PCE)       |            |                                          |  |
   |  +------|------+        +--^-------------|--+            +------|----------------------------|------+  |
   +---------|------------------|-------------|----------------------|----------------------------|---------+
             | Encapsulation    | LinkState   | SR Policy            | Enable/Disable             | Enable/Disable
             | Policy           |             |                      | the Service Segment        | the Service Segment
             | (per-flow)       |             |                      | (End.AN SID:S1)            | (End.AN SID:S2)
   +---------|------------------|-------------|----------------------|----------------------------|---------+
   |  +------v------------------|-------------v--+            +------v------+              +------v------+  |
   |  |                                          |            |SRv6 Service |              |SRv6 Service |  |
   |  |            SRv6 SR Source Node           |------------|Function Node|--------------|Function Node|  |
   |  |                                          |            |    (S1)     |              |    (S2)     |  |
   |  +------------------------------------------+            +-------------+              +-------------+  |
   +--------------------------------------------- SRv6 domain ----------------------------------------------+
~~~
{: #in-network-sfc-control-plane title="In-network SFC (Control Plane)"}

The In-network SFC Controller consists of the following three components:

* Service Function Manager: This component is responsible for defining the state and SID of network service functions on an SRv6 Service Function Node and managing the Service Segment.
* SRv6 Policy Manager: This component generates SR Policies that fulfill SFC/QoS requirements from the headend to the tailend and sends them to the SRv6 SR Source Node.
* Encapsulation Policy Manager: This component generates an Encapsulation Policy that corresponds to a specific flow and SR Policy, and sends them to the SRv6 SR Source Node.

## Service Function Manager
The Service Function Manager is responsible for enabling and disabling network service functions on a specific SRv6 Service Function Node.
To manage service segments, it leverages the extensions provided in BGP-LS, as outlined in [I-D.draft-watal-idr-bgp-ls-srv6-sfc-enabler], and defines the following parameters:

* Behavior: End.AN
* SID: The SID of End.AN (in IPv6 Address format). Service segments that support slicing are specified here as Flex-Algo SIDs.
* Function Name: Type of network service function
* Action: Enable
* TLV:
    * Specification of the Anycast Segment Group: When deploying multiple Network Functions within the same context, it SHOULD to use the Anycast Group TLV to specify the same anycast segment group SID.
    * Allows for the specification of unique parameters and context associated with a particular network service function.

## SR Policy Manager
The SR Policy Manager is implemented as an SRv6 Active Stateful Path Computation Element (PCE).
It acquires the Traffic Engineering Database (TED) of the SRv6 domain using BGP-LS and deploys SR Policies via PCEP or BGP SR Policy.

The SR Policy can utilize CSPF to satisfy various requirements, including SFC and QoS.
Moreover, SR Policies can be defined on a per-flow or per-TE basis, providing flexibility.

## Encapsulation Policy Manager
For communication with each node, an extended protocol based on BGP Flow Spec is used for SR Policy.
SR Policy specification may consist of three components: endpoint, color, and policy name (MAY).

The set of endpoint and color is transmitted as described in {{!I-D.draft-ietf-idr-ts-flowspec-srv6-policy}}.
Furthermore, to associate the Policy name with flows, [I-D.draft-skyline-idr-flowspec-srv6-policy-per-flow] is employed.

## Other Managers
Additional managers that can be added to the In-network SFC Controller may include:

* App Metric Manager: Collect metrics to evaluate SRv6 Policy, including SFC/QoS.
* Hypervisor Resource Manager: Managing aspects like memory usage on the hypervisor that provides VNFs and monitors the available resources.
* Service Network Function Deployer: Handles the deployment of VNFs to SRv6 Service Function Nodes.

The metrics gathered by these other managers can be utilized as computational inputs by the various managers described in this document.
Details regarding each specific manager are outside the scope of this document.

# Manageability Considerations
SR-aware services are defined in {{!I-D.draft-ietf-spring-sr-service-programming}}, and the pseudocode for End.AN is provided in this document.

The BGP-LS Service Segment is defined in [I-D.draft-watal-idr-bgp-ls-srv6-sfc-enabler].
As mentioned in Section 5.1, it SHOULD use the Anycast Group TLV when employing the network service function as an Anycast Segment.

The BGP FlowSpec with SRv6 Policy is defined in {{!I-D.draft-ietf-idr-ts-flowspec-srv6-policy}} and [I-D.draft-skyline-idr-flowspec-srv6-policy-per-flow].
As described in Section 5.3, it SHOULD be used for each Service Function Chain in a way that allows coexistence with other network contexts, such as slicing, VPNs, and areas.

# Security Considerations
This document does not introduce any new security vulnerabilities.
However, it should be noted that in In-network SFC, the SRv6 Service Function Node itself is globally accessible via IPv6.

The security requirements and mechanisms described in [RFC8402], [RFC8754], and [RFC8986] are also applicable to this document.

# IANA Considerations
Ask IANA for viable.
This document has no IANA actions.

Value      Description                         Reference
-----------------------------------------------------------------------------------------
TBA1-1     End.AN - SR-aware function (native) {{!I-D.draft-ietf-spring-sr-service-programming}}

# Implementation Status
TODO: Remove this section before publishing.

## Data Plane
TODO

## Control Plane
TODO

--- back

# Acknowledgments
{:numbered="false"}
The authors would like to acknowledge the review and inputs from Mitsuru Maruyama, Katsuhiro Sebayashi, Yuma Ito, Taisei Tanabe, and Kouta Aoki.
