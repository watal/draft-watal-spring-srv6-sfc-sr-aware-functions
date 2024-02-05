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

SRv6 SFC realizes traffic steering through various ordered sets of network functions.

This document describes the concept of "SRv6 In-network SFC Architecture" that realizes comprehensive manage of SFC with SRv6-aware network function.
This architecture provides programmability for SRv6 operators to manage SFC through provisioning network functions as SRv6 Segment, collecting network states, and applying Service Function Chain or TE Policy based on a user's demand.

To realize SRv6 In-network SFC, D-Plane/C-Plane components are required as follows:

* D-Plane:
   * SRv6-aware network service functions: "End.AN" behavior that is described in {{!I-D.draft-skyline-spring-srv6-aware-services}}.
* C-Plane: To comprehensively manage SRv6 SFCs, a controller with the following functions is used.
   * Enabling End.AN: activates network service functions at SR segment endpoint nodes.
   * Adding SR Policy: provisions service function chains at SR source nodes.
   * Adding encapsulation policy: classifies the target flow and provisions encapsulation policy at SR source nodes.

--- middle

# Introduction
Segment Routing over IPv6 (SRv6) {{!RFC8986}} enables packet steering through a set of instructions called a segment list.
Each SR segment endpoint node can provide SRv6 Endpoint Behaviors, such as Node/Adjacency Segments, VPNs, and Binding Segments.

Service Function Chaining (SFC) {{!RFC7665}} realizes various use cases (e.g. FW, IPS/IDS, NAT, and DPI).
In the current SRv6 architecture, SFC proxies like End.AS/AD/AM are necessary to apply network functions.

To manage an SRv6 network, several protocols are defined:

* SR Policy management: PCEP (RFC 5440), BGP SR Policy {{!I-D.draft-ietf-idr-segment-routing-te-policy}}
* Encapsulation policy management: BGP Flowspec {{!I-D.draft-ietf-idr-ts-flowspec-srv6-policy}}
* Network function advertisement: BGP-LS Service Segment {{!I-D.draft-ietf-idr-bgp-ls-sr-service-segments}}

To reduce operational costs and achieve advanced use cases, we aim for SRv6 native SFC and comprehensive management of SRv6 networks.
To enable advanced programmability for network operators, network functions in the SRv6 domain must be manageable as Segments, and SFC/QoS services must be manageable on a per-flow basis.

To achieve this, the following technologies are needed:

* SRv6-aware network functions
  * End.AN is already defined in {{!I-D.draft-ietf-spring-sr-service-programming}}, but there is no specification and implementation.
* The ability to control SR-aware network functions from a controller.
  * The controller can obtain the state of the Service Segments, but there is no protocol to manage them.

This document defines an architecture for SRv6 In-network SFC using SRv6-based methods, covering both the data plane (D-Plane) and control plane (C-Plane).
The generic SFC architecture is covered in {{!RFC7665}}, while the SFC architecture based on Segment Routing is covered in {{!I-D.draft-li-spring-sr-sfc-control-plane-framework}}, and therefore outside the scope of this document.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all capitals, as shown here.

# Terminology
This document leverages the terminology defined in {{!RFC7665}}, {{!RFC8402}}, {{!RFC8754}}, {{!RFC8986}} and {{!RFC9256}}. It also introduces the following new terms.

* SRv6 Service Function Node: An SR segment endpoint node that provides SRv6-aware network functions as a service segment.

# Design Objectives
## Goals/Objectives
SRv6 In-network SFC Architecture is designed to provide programmability of SRv6 native SFC for operators.
This programmability includes provisioning service function chains, managing SR Policy based on collected LinkState and network metrics, and traffic steering of each flow for applying SFC and SLA assurance.

This architecture realizes the following advantages by using SRv6-aware network functions:
* Efficient forwarding using SRv6 Service Function Node located in the SR domain.
* Network function management, redundancy, and protection using the SRv6 ecosystem.
* Comprehensive management of SRv6 networks, including SRv6-aware network functions, Service Function Chain, per-flow TE, and network metrics.

## Requirements
To achieve these objectives, SRv6 In-network SFC is based on several key requirements:

1. In-network Processing: packet forwarding using network functions within an SRv6 domain.
2. Deployment of Network Service Functions to Any Nodes: deploying functions to any nodes based on user demand.
3. Integration with Traffic Engineering (TE): simultaneously representing SFC policy and QoS policy with SR Policy.
4. Per-Flow Identification: to apply SFC Policies per flow, classifying at the SRv6 SR source node based on the 5-tuple or even finer granularity.
5. Centralized Management: managing the entire SRv6 domain by the controller, including aggregating network service functions on the network and per-flow encapsulation policies.
6. Ease of Deployment: Employs existing protocols commonly used in controllers, such as BGP and PCEP.
7. Coexistence with Existing Services: Operating while maintaining core network services like slicing and VPNs.

# Overview of SRv6 In-Network SFC Architecture
In Figure 1 and Figure 2, the Overview of SRv6 In-network SFC and Current SRv6 SFC are shown, respectively.

~~~ drawing
   +------------------------------------ SRv6 In-network SFC Controller ------------------------------------+
   |  +-------------+        +-------------------+            +------------------------------------------+  |
   |  |Encapsulation|        |     SR Policy     |            |                                          |  |
   |  |   Policy    |        |      Manager      |            |         Service Function Manager         |  |
   |  |   Manager   |        |       (PCE)       |            |                                          |  |
   |  +------|------+        +--^-------------|--+            +------|----------------------------|------+  |
   +---------|------------------|-------------|----------------------|----------------------------|---------+
             |                  |             |                      |                            |
             |                  |             |                      |
             |                  |             |                      |                            |
   +---------|------------------|-------------|----------------------|----------------------------|---------+are
   |  +------v------------------|-------------v--+            +------v------+              +------v------+  |
   |  |                                          |            |SRv6 Service |              |SRv6 Service |  |
   |  |            SRv6 SR Source Node           |------------|Function Node|--------------|Function Node|  |
   |  |                                          |            |             |              |             |  |
   |  +------------------------------------------+            +-------------+              +-------------+  |
   +--------------------------------------------- SRv6 domain ----------------------------------------------+
~~~
{: #srv6-in-network-sfc title="Overview of SRv6 In-network SFC"}

~~~ drawing
   +--------------------------------------------------------------------------------------------------------+
   |  +------------------------------------------+            +-------------+              +-------------+  |
   |  |                                          |            |             |              |             |  |
   |  |            SRv6 SR Source Node           |------------|  SFC Proxy  |--------------|  SFC Proxy  |  |
   |  |                                          |            |             |              |             |  |
   |  +------------------------------------------+            +---|-----^---+              +---|-----^---+  |
   +--------------------------------------------- SRv6 domain ----|-----|----------------------|-----|------+
                                                                  |     |                      |     |
                                                              +---v-----|---+              +---v-----|---+
                                                              | SR-unaware  |              | SR-unaware  |
                                                              |   Network   |              |   Network   |
                                                              |  Function   |              |  Function   |
                                                              +-------------+              +-------------+
~~~
{: #current-srv6-sfc title="Current SRv6 SFC"}

In-network SFC provides services inside the SR domain by using the SR-aware network function.
This eliminates forwarding by the SFC Proxy and improves forwarding efficiency compared to the current SRv6 SFC.

This architecture allows the SRv6-aware function to leverage the SRv6 ecosystem, allowing FRR and other protection and anycast mechanisms to be used without modification. Thus, high fault tolerance and SRv6 native redundancy can be achieved.

In addition, the In-network SFC Architecture enables comprehensive management of SRv6 SFCs by the SRv6 In-network SFC Controller. This enables management of SRv6-aware network functions as Service Segments, construction and per-flow provisioning of Service Function Chains, and LinkState and metric collection for path calculation.

# Data Plane
The Data Plane is designed as follows to satisfy requirements 1, 3, and 4:

* End.AN-based Service Segment Provisioning: To achieve in-network processing with SRv6, the data plane utilizes End.AN to handle SR-aware network service functions.
* SRv6 Policy: Achieving SFC and QoS requirements through the Segment List.
* Per-Flow Encapsulation Policy: Applying per-flow requirements through the Encapsulation Policy using PBR.

{{!RFC7665}} outlines a procedure in which each packet is classified by the Service Classification Function, then forwarded to the Service Function Forwarder, and subsequently delivered to a specific network service function.
In the SRv6 native SFC architecture, the SRv6 SR source node classifies the flow and forwards it to a specific SRv6 Service Function Node by specifying a Segment List that represents a particular Service Function Chain.

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
{: #in-network-sfc-data-plane title="SRv6 In-network SFC (Data Plane)"}

Figure 3 shows an example of SFC with two network service functions.
Firstly, the SRv6 SR source node classifies the flow and encapsulates it with an SRH containing the segment list <s1, s2>.
Next, the SRv6 Service Function Node for s1 receives the packet and applies End.AN.
Finally, the SRv6 Service Function Node for s2 receives the packet and also applies End.AN, thus achieving SFC.

Deploying multiple instances of the same network service function in an SRv6 domain enables the implementation of Service Function Chaining as a multipath.
In such scenarios, stateful network service functions like FW or NAT MUST establish state-sharing mechanisms among SRv6 Service Function Nodes.
Additionally, Service Segments that provide stateless network service functions can achieve geographically efficient delivery by utilizing anycast SIDs.

## End.AN-based Service Segment Provisioning
By utilizing End.AN at an SR segment endpoint node, End.AN can be realized for providing Service Segments natively in SRv6. Simultaneously, this enables the provisioning of network service functions at any location based on customer demand within the SR domain.

Functions with the same role MAY be assigned as the same Service Segment within the SR domain.
By using Anycast SIDs, multiple nodes can be grouped as part of the same Service Segment.

End.AN MAY have optional arguments that can be passed as parameters to bound network service functions.

### Anycast Segment
The concept of the Anycast Segment is introduced in {{!RFC8402}}. It is permissible to configure the same network service function segment as the same Anycast segment.
In such cases, the state between network service functions MUST be shared mutually.

### When a Network Service Function Goes Down
If a network service function experiences a failure, the associated route MUST be promptly removed. In the case of Anycast configuration, it MUST be gracefully rerouted to other nodes.
Additionally, if no alternative nodes are available, consider either dropping the packet and sending an ICMP Destination Unreachable message or forwarding it as a pass-through.

### Fast ReRoute
Because Service Function Chains are structured as a Segment List, the order of application is guaranteed even in the event of Fast ReRoute for functions.
In such cases, if Anycast Segments are used, it is permissible to take a detour to a more optimal node.

## SRv6 Policy
In SRv6 In-network SFC, each Service Function Chain is represented as an SRv6 Policy {{!RFC9256}}.
The purpose or intent of each SRv6 Policy can be identified using attributes such as color or name.

## Per-Flow Encapsulation Policy
In the SRv6 SR source node, which serves as the Service Classifier, packets are classified on a per-flow basis using PBR and encapsulated with SRv6 Policy.
Therefore, the SRv6 SR source node MUST be capable of identifying packets using at least a 5-tuple or even more detailed information.

# Control Plane
The Control Plane is designed as follows to satisfy requirements 2, 3, 4, 5, 6, and 7:

* Network Service Functions: Enable/Disable network service functions on any node within the SRv6 domain.
* SRv6 Policy: Calculate constrained paths that achieve both SFC and QoS requirements.
* Per-flow Encapsulation Policy: Classify flows and issue encapsulation policies to achieve per-flow SFC.
* SDN Approach: Centralized management of network service functions, encapsulation policies, and SR Policies by the controller.
* Generic Protocol: Utilizing standardized protocols such as MP-BGP and PCEP, with minimal extensions.
* Integration with Current Network Contexts: Policy identification methods that coexist with existing network contexts, including SR Policy Colors associated with slices, VPNs, and more.

~~~ drawing
   +------------------------------------ SRv6 In-network SFC Controller ------------------------------------+
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
{: #in-network-sfc-control-plane title="SRv6 In-network SFC (Control Plane)"}

The SRv6 In-network SFC Controller consists of the following three components:

* Service Function Manager: This component is responsible for defining the state and SID of network service functions on an SRv6 Service Function Node and managing the Service Segment.
* SRv6 Policy Manager: This component generates SR Policies that fulfill SFC/QoS requirements from the headend to the tailend and sends them to the SRv6 SR source node.
* Encapsulation Policy Manager: This component generates an Encapsulation Policy that corresponds to a specific flow and SR Policy, and sends them to the SRv6 SR source node.

## Service Function Manager
The Service Function Manager is responsible for enabling and disabling service segments of SRv6 Service Function Nodes.
To manage service segments, it utilizes the extensions provided in BGP-LS, as outlined in [I-D.draft-watal-idr-bgp-ls-srv6-sfc-enabler], and defines the following parameters:

* Behavior: End.AN
* SID: The SID of End.AN (in IPv6 Address format). Service segments that support slicing are specified here as Flex-Algo SIDs.
* Function Name: Type of network service function
* Action: Enable
* TLV:
    * Specification of the Anycast Segment Group: When deploying multiple Network Functions within the same context, it MUST use the Anycast Group TLV to specify the same anycast segment group SID.
    * Allows for the specification of unique parameters and context associated with a particular network service function.

## SR Policy Manager
The SR Policy Manager is implemented as an SRv6 Active Stateful Path Computation Element (PCE).
It acquires the Traffic Engineering Database (TED) of the SRv6 domain using BGP-LS and deploys SR Policies via PCEP or BGP SR Policy.

The SR Policy can utilize CSPF to satisfy various requirements, including SFC and QoS.
Moreover, SR Policies can be defined on a per-flow or per-TE basis, providing flexibility.

## Encapsulation Policy Manager
For communication with each node, an extended protocol based on BGP Flow Spec is used for SR Policy.
SR Policy specification consists of three components: endpoint, color, and policy name.

The set of endpoints and color is transmitted as described in {{!I-D.draft-ietf-idr-ts-flowspec-srv6-policy}}.
Furthermore, to associate the Policy name with flows, [I-D.draft-skyline-idr-flowspec-srv6-policy-per-flow] is employed.

## Other Managers
Additional managers that can be added to the SRv6 In-network SFC Controller MAY include:

* Metric Manager (L3/L4/L7): Collect metrics to evaluate SRv6 Policy, including SFC/QoS. e.g. SRv6 Path Tracing, IPFIX, TCP statistics.
* Hypervisor Resource Manager: Managing aspects like memory usage on the hypervisor that provides network functions and monitors the available resources.
* Service Network Function Deployer: Handles the deployment of network functions to SRv6 Service Function Nodes.

The metrics collected by these other managers can be used as inputs for managers described in this document.
Details regarding each specific manager are outside the scope of this document.

# Security Considerations
It has to be noted the SRv6 Service Function Node is globally accessible with IPv6.
If a network function has a security vulnerability, this node or other device on the IPv6 network may be attacked.

The security requirements and mechanisms described in [RFC8402], [RFC8754], and [RFC8986] are also applicable to this document.

# IANA Considerations
This document has no IANA actions.

--- back

# Appendix A. Highly Reliable Firewall Service Using SRv6 End.AN
If you implement a firewall as a SRv6-aware function at an SRv6 End.AN node, you can forward packets using anycast and also achieve 'Fast Reroute'.
This makes clustering firewall easier as well.


# Appendix B. Flexible & Low-latency Remote Production Service
In the context of video remote production, you can perform video processing within a SRv6 network by combining multiple network functions (SFC).
If you have to distribute multiple connections from several source, you can also use multicast packet in the SRv6 network.


# Acknowledgments
{:numbered="false"}
The authors would like to acknowledge the review and inputs from Mitsuru Maruyama, Katsuhiro Sebayashi, Yuma Ito, and Taisei Tanabe.
We partially obtained the research results from NICT’ s commissioned research No. JPJ012368C03101.
