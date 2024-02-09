---
title: "SRv6 Native SFC Architecture"
abbrev: "SRv6 Native SFC"
docname: draft-watal-spring-srv6-native-sfc-latest
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

This document describes the architecture of SRv6 native SFC which enables comprehensive management of SFC with SRv6-aware network functions.

This architecture provides the following benefits:
* Simple architecture designed for all devices and protocols to use SRv6
* Low latency and Reduced capital expenditure (CAPEX) with proxy-free SFC
* Scalable control plane and Reduced operating expense (OPEX) by centralized management approach
* Enhance programmability through centralized control of TEs including SFC and QoS and the use of user-defined network functions

XXX: In-networkというワードが特に新しいのでそこを推しているかのようなタイトルになっている．一番与えたい印象や新しいポイント嬉しいポイントが明確になったら再度タイトルについて検討する → 利点を箇条書きで示した
XXX: CAPEXみたいなマーケットに出た時にどうなるかわからない信憑性の薄いことを主要メリットとして挙げるのではなく，よりシンプルなアーキテクチャでビルディングブロックが減る，みたいな誰がみても明らかに真な理由を書く．
XXX: low latencyは我々が実験したところ，もちろん削減方向にはなるが，全く気にするに値しないレベルなので，主要なメリットの一番として挙げるには明らかに不適

--- middle

# Introduction
Segment Routing over IPv6 (SRv6) {{!RFC8986}} enables packet steering through a set of instructions called a segment list.
Each SR segment endpoint node provides SRv6 Endpoint Behaviors, including Prefix/Adjacency-Segments, VPNs, and Binding Segments.

Service Function Chaining (SFC) {{!RFC7665}} can be used in various situations (e.g. FW, IPS/IDS, NAT, and DPI).
Within the current SRv6 architecture, SFC proxies like End.AS/AD/AM are necessary to apply network functions.
In addition, the SFC architecture based on Segment Routing is described in {{!I-D.draft-li-spring-sr-sfc-control-plane-framework}}.

This document defines SRv6 native SFC architecture, which aims

to provides the following benefits:

To realize SRv6 Native SFC, D-Plane/C-Plane components are required as follows:
* D-Plane: utilizes SRv6-aware functions to minimize forwarding nodes and utilizes the SRv6 ecosystem for redundancy and protection. XXX: D-Planeのもたらす利点を整理して記入 → done
  * SRv6-aware network service functions: employing the "End.AN" behavior as described in {{!I-D.draft-skyline-spring-srv6-aware-services}}.
* C-Plane: uses SRv6 Native Function Controller to provide programmability to SRv6 network operators by establishing SFCs and manipulating SRv6 Service Function Nodes.
  * Enabling End.AN: activates network service functions at SR segment endpoint nodes.
  * Adding SR Policy: provisions service function chains at SR source nodes.
  * Applying SR Policy per flow: classifies the target flow and adopts SR policy at SR source nodes.

# Terminology

## Related RFCs and Internet-Drafts

* {{!RFC7665}} describes the SFC architecture and defines the following terms: SFC, SFC Proxy, and service classification function.
* {{!RFC8402}} describes the Segment Routing architecture and defines the following terms: Segment Routing (SR), SR Domain, Segment ID (SID), SRv6, SR Policy, Prefix segment, Adjacency segment, and Anycast segment.
* {{!RFC8754}} describes the encoding of IPv6 segments in the SRH and defines the following terms: SR source node, transit node, and SR segment endpoint node.
* {{!RFC8986}} describes the main SRv6 behaviors and defines the following terms: SRv6 SID function and SRv6 Endpoint behavior.
* {{!RFC9256}} describes the SR Policy architecture.
* {{!RFC9522}} describes the principle of internet traffic engineering and defines the following terms: egress node, ingress node, metric, measurement methodology, provisioning, Quality of Service (QoS), Service Level Agreement (SLA), and Traffic-engineering system.
* {{!I-D.draft-ietf-spring-sr-service-programming}} describes and defines the following terms: service segment, SR-aware service, SR-unaware Service, End.AS, End.AD and End.AM.

## Newly Defined Terminology

The following terms are used in this document as defined below:

* SRv6 Service Function Node: an SR segment endpoint node that offers SRv6-aware network functions as service segments.
* Native SFC: provides SFC within the SR domain by using SR-aware network functions.

## Requirements Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all capitals, as shown here.

# Design Objectives and Requirements
## Goals/Objectives

SRv6 in-network SFC architecture is designed to enhance the capabilities of service provisioning with SFC.

This architecture for aims the following objectives:

* Reducing latency and CAPEX (CAPital EXpenditure). XXX: どういうふうにefficientなのかを明確に説明する → 説明した
   * Reduce network entity and wasting address.
   * Realize more simple architecture through no longer depending on SFC Proxy for SR-Unaware Functions.
   * Provide users with additional programmability through abstraction with End.AN.
* Providing programmability for SRv6 operators to deliver SFC and other network services.
   * Handle network functions in SRv6 segments, collecting network states, and applying SFCs or TE Policies based on user demand.
   * Also, this architecture provides programmability including provisioning service function chains, managing SR Policy based on collected LinkState and network metrics, and traffic steering of each flow for applying SFC and SLA assurance.
   * Comprehensive management of SRv6 networks, including SRv6-aware network functions, SFC, per-flow TE, and network metrics.
* Ease of Deployment and Complete backward compatibility: leverage the SRv6 ecosystem, including SR forwarding, redundancy, protection, etc.
   * No additional implementation is required, just a combination of existing protocols.
   * Coexist without disrupting existing SRv6 network services such as TE and VPN.

XXX: realizeに対応する言葉としてmanagement/redundancy/protectionはおかしい．using SRv6 ecosystemも言葉足らずで，既存のSRv6標準やMPLSでは実現できなかったことが, 我々の提案する"THE"SRv6-aware functionもしくはEnd.ANを用いることで実現できる．的なことを書く → objective としてまとめ直し，また management はアプローチなので削除した．SR ネットワークを管理し，オペレーショナルコストを下げるのが目的．

XXX: managementは上のと重複しているので，具体的な説明こみでmanagementは1つにする / この文自体そもそも, SRv6-aware network functions(End.AN)が実現するものではなく，C-Planeの整理と提案によって実現されるメリットなのでここに書くのは不適当 → 削除した

## Requirements
To achieve these objectives, SRv6 Native SFC is based on several key requirements:

1. Native Processing: packet forwarding using network functions within an SRv6 domain.
2. Provide Programmability: establish service function chains according to user demand and provide additional programmability through abstraction of SRv6 behavior.
3. Centralized Management: managing the entire SRv6 domain by the controller, including aggregating network service functions on the network and per-flow encapsulation policies.
4. Manipulation of Service Segment: deploying service segment to an SRv6-aware function node.
5. Per-Flow Identification: to apply SFC Policies per flow, classifying at the SRv6 SR source node based on the 5-tuple or even finer granularity.
6. Straightforward Extension: employs existing protocols commonly used in controllers, such as BGP and PCEP, and coexists with existing SRv6 network Services like slicing and VPNs.

# Overview of SRv6 Native SFC Architecture
In Figure 1 and Figure 2, the Overview of SRv6 Native SFC and Current SRv6 SFC are shown, respectively.

~~~ drawing
 +-------------- SRv6 Native SFC Controller --------------+
 | +---------+ +--------------+ +---------------------------+ |
 | |  Encap  | |  SR Policy   | |                           | |
 | | Policy  | |   Manager    | |  Service Function Manage  | |
 | | Manager | |    (PCE)     | |                           | |
 | +----|----+ +-^----------|-+ +------|--------------|-----+ |
 +------|--------|----------|----------|--------------|-------+
        |        |          |          |              |
 +------|--------|----------|----------|--------------|-------+
 | +----v--------|----------v-+ +------v-----+ +------v-----+ |
 | |                          | |SRv6 Service| |SRv6 Service| |
 | |    SRv6 SR Source Node   |-|  Function  |-|  Function  | |
 | |                          | |    Node    | |    Node    | |
 | +--------------------------+ +------------+ +------------+ |
 +----------------------- SRv6 domain ------------------------+
~~~
{: #srv6-in-network-sfc title="Overview of SRv6 Native SFC"}

~~~ drawing
 +------------------------------------------------------------+
 | +--------------------------+ +------------+ +------------+ |
 | |                          | |            | |            | |
 | |    SRv6 SR Source Node   |-|  SFC Proxy |-|  SFC Proxy | |
 | |                          | |            | |            | |
 | +--------------------------+ +---|-----^--+ +---|-----^--+ |
 +---------- SRv6 domain -----------|-----|--------|-----|----+
                                    |     |        |     |
                                +---v-----|--+ +---v-----|--+
                                | SR-unaware | | SR-unaware |
                                |   Network  | |   Network  |
                                |  Function  | |  Function  |
                                +------------+ +------------+
~~~
{: #current-srv6-sfc title="Current SRv6 SFC"}

Native SFC provides services within an SR domain by using the SR-aware network function.
This eliminates forwarding by the SFC Proxy and improves forwarding efficiency compared to the current SRv6 SFC.

This architecture allows the SRv6-aware function to leverage the SRv6 ecosystem, allowing FRR and other protection and anycast mechanisms to be used without modification. Thus, high fault tolerance and SRv6 native redundancy can be achieved.

In addition, the Native SFC architecture enables comprehensive management of SRv6 SFCs by the SRv6 Native SFC Controller. This enables management of SRv6-aware network functions as Service Segments, construction and per-flow provisioning of SFCs, and LinkState and metric collection for path calculation.

# Data Plane
The Data Plane is designed as follows:

* Provide SRv6-aware network service functions: to achieve in-network processing with SRv6, the data plane utilizes End.AN to handle SR-aware network service functions.
* Represent the service function chain as an SR Policy: achieving SFC and QoS requirements through the Segment List.
* Applying SR Policy per flow: classifies the target flow and adopts SR policy at SR source nodes using PBR.
* Allow user-defined behavior extensions: allows user-defined functions using End.AN to improve the programmability of SRv6 network services. Abstraction of behavior implementation using AN reduces implementation costs compared to user-defined behavior.

XXX: End.ANのfunctionの内容はbase-set|main-setとして考えられるFW/IPS/IDS/NAT/DPI以外にもuser-defined(RFC8986で記載)なfunctionsを定義する余地を入れ込みたい(e.g. Video Processing), like End.AN.VideoPinP, → service segment としての抽象化と，それによるプログラマビリティの向上について書いてみた．

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
{: #in-network-sfc-data-plane title="SRv6 Native SFC (Data Plane)"}

Figure 3 shows an example of SFC with two network service functions.
Firstly, the SRv6 SR source node classifies the flow and encapsulates it with an SRH containing the segment list <S1, S2>.
Next, the SRv6 Service Function Node for S1 receives the packet and applies End.AN.
Finally, the SRv6 Service Function Node for S2 receives the packet and also applies End.AN, thus achieving SFC.

Deploying multiple instances of the same network service function in an SRv6 domain enables the implementation of SFC as a multipath.
In such scenarios, stateful network service functions like FW or NAT MUST establish state-sharing mechanisms among SRv6 Service Function Nodes.
Additionally, Service Segments that provide stateless network service functions can achieve geographically efficient delivery by utilizing Anycast-SIDs.

## End.AN-based Service Segment Provisioning
By utilizing End.AN at an SR segment endpoint node, End.AN can be realized for providing Service Segments natively in SRv6. Simultaneously, this enables the provisioning of network service functions at any location based on customer demand within the SR domain.

Functions with the same role MAY be assigned as the same Service Segment within the SR domain.
By using Anycast-SIDs, multiple nodes can be grouped as part of the same Service Segment.

End.AN MAY have optional arguments that can be passed as parameters to bound network service functions.

### Anycast Segment
The concept of the Anycast segment is introduced in {{!RFC8402}}. It is permissible to configure the same network service function segment as the same Anycast segment.
In such cases, the state between network service functions MUST be shared mutually.

### When a Network Service Function Goes Down
If a network service function experiences a failure, the associated route MUST be promptly removed. In the case of Anycast configuration, it MUST be gracefully rerouted to other nodes.
Additionally, if no alternative nodes are available, consider either dropping the packet and sending an ICMP Destination Unreachable message or forwarding it as a pass-through.

### Fast ReRoute
Because SFCs are structured as a Segment List, the order of application is guaranteed even in the event of Fast ReRoute for functions.
In such cases, if Anycast segments are used, it is permissible to take a detour to a more optimal node.

## SRv6 Policy
In SRv6 Native SFC, each SFC is represented as an SRv6 Policy {{!RFC9256}}.
The purpose or intent of each SRv6 Policy can be identified using attributes such as color or name.

## Per-Flow Encapsulation Policy
In the SRv6 SR source node, which serves as the Service Classifier, packets are classified on a per-flow basis using PBR and encapsulated with SRv6 Policy.
Therefore, the SRv6 SR source node MUST be capable of identifying packets using at least a 5-tuple or even more detailed information.

# Control Plane
The Control Plane is designed as follows:

* Network Service Functions: enable/disable network service functions on any node within the SRv6 domain.
* SRv6 Policy: calculate constrained paths that achieve both SFC and QoS requirements.
* Per-flow Encapsulation Policy: classify flows and issue encapsulation policies to achieve per-flow SFC.
* SDN Approach: centralized management of network service functions, encapsulation policies, and SR Policies by the controller.
* Generic Protocol: utilizing standardized protocols such as MP-BGP and PCEP, with minimal extensions.
* Integration with Current Network Contexts: policy identification methods that coexist with existing network contexts, including SR Policy Colors associated with slices, VPNs, and more.

~~~ drawing
 +-------------- SRv6 Native SFC Controller --------------+
 | +---------+ +--------------+ +---------------------------+ |
 | |  Encap  | |  SR Policy   | |                           | |
 | | Policy  | |   Manager    | | Service Function Manager  | |
 | | Manager | |    (PCE)     | |                           | |
 | +----|----+ +-^----------|-+ +-------|-------------|-----+ |
 +------|--------|----------|-----------|-------------|-------+
        |        |          |           |             |
      Encap  LinkState  SR Policy  Enable/Disable     |
     Policy  (BGP-LS)  (PCEP/BGP) a Service Segment   |
  (BGP Flowspec) |          |     (End.AN SID:S1)  (SID:S2)
        |        |          |           |             |
 +------|--------|----------|-----------|-------------|-------+
 | +----v--------|----------v-+ +------v-----+ +------v-----+ |
 | |                          | |SRv6 Service| |SRv6 Service| |
 | |    SRv6 SR Source Node   |-|  Function  |-|  Function  | |
 | |                          | |    Node    | |    Node    | |
 | +--------------------------+ +------------+ +------------+ |
 +----------------------- SRv6 domain ------------------------+
~~~
{: #in-network-sfc-control-plane title="SRv6 Native SFC (Control Plane)"}

The SRv6 Native SFC Controller consists of the following three components:

* Service Function Manager: this component is responsible for defining the state and SID of network service functions on an SRv6 Service Function Node and managing the Service Segment.
* SRv6 Policy Manager: this component generates SR Policies that fulfill SFC/QoS requirements from the headend to the tailend and sends them to the SRv6 SR source node.
* Encapsulation Policy Manager: this component generates an Encapsulation Policy that corresponds to a specific flow and SR Policy, and sends them to the SRv6 SR source node.

## Service Function Manager
The Service Function Manager is responsible for enabling and disabling service segments of SRv6 Service Function Nodes.
To manage service segments, it utilizes the extensions provided in BGP-LS Service Segment, as outlined in {{!I-D.draft-ietf-idr-bgp-ls-sr-service-segments}} and {{!I-D.draft-watal-idr-bgp-ls-srv6-sfc-enabler}}, and defines the following parameters:

* Behavior: End.AN
* SID: the SID of End.AN (in IPv6 Address format). Service segments that support slicing are specified here as Flex-Algo SIDs.
* Function Name: type of network service function
* Action: enable
* TLV:
    * Specification of the Anycast Segment Group: when deploying multiple Network Functions within the same context, it MUST use the Anycast Group TLV to specify the same anycast segment group SID.
    * Allows for the specification of unique parameters and context associated with a particular network service function.

## SR Policy Manager
The SR Policy Manager is implemented as an SRv6 Active Stateful Path Computation Element (PCE).
It acquires the Traffic Engineering Database (TED) of the SRv6 domain using BGP-LS and deploys SR Policies via PCEP {{!RFC5440}} or BGP SR Policy {{!I-D.draft-ietf-idr-segment-routing-te-policy}}.

The SR Policy can utilize CSPF to satisfy various requirements, including SFC and QoS.
Moreover, SR Policies can be defined on a per-flow or per-TE basis, providing flexibility.

## Encapsulation Policy Manager
For communication with each node, an extended protocol based on BGP Flow Spec is used for SR Policy.
SR Policy specification consists of three components: endpoint, color, and policy name.

The set of endpoints and color is transmitted as described in {{!I-D.draft-ietf-idr-ts-flowspec-srv6-policy}}.

## Other Managers
Additional managers that can be added to the SRv6 Native SFC Controller MAY include:

* Metric Manager (L3/L4/L7): collect metrics to evaluate SRv6 Policy, including SFC/QoS. e.g. SRv6 Path Tracing, IPFIX, TCP statistics.
* Hypervisor Resource Manager: managing aspects like memory usage on the hypervisor that provides network functions and monitors the available resources.
* Service Network Function Deployer: handles the deployment of network functions to SRv6 Service Function Nodes.

The metrics collected by these other managers can be used as inputs for managers described in this document.
Details regarding each specific manager are outside the scope of this document.

# Security Considerations
It has to be noted the SRv6 Service Function Node is globally accessible with IPv6.
If a network function has a security vulnerability, this node or other device on the IPv6 network may be attacked.

The security requirements and mechanisms described in {{!RFC8402}}, {{!RFC8754}}, and {{!RFC8986}} are also applicable to this document.

# IANA Considerations
This document has no IANA actions.

--- back

# Highly Reliable Firewall Service Using SRv6 End.AN
If you implement a firewall as an SRv6-aware function at an SRv6 End.AN node, you can forward packets using anycast SID and also achieve TI-LFA Fast Reroute {{!I-D.draft-ietf-rtgwg-segment-routing-ti-lfa}}.
This makes clustering firewalls easier as well.

# Flexible and Low-latency Remote Production Service
In the context of video remote production, you can perform video processing within an SRv6 network by combining multiple network functions (SFC).
If you have to distribute multiple connections from several sources, you can also use multicast packets in the SRv6 network.

# XXX: SRv6-aware NAT
In the Interop Tokyo 2023 shownet's backbone SRv6 network, they had to decapsulate packets to conduct Network Address Translation.
If you use SRv6-aware NAT, you don't have to decap the packets when traversing the NAT function.
This contributes to achieving a simpler network architecture(design?)

# Acknowledgments
{:numbered="false"}
The authors would like to acknowledge the review and inputs from Mitsuru Maruyama, Katsuhiro Sebayashi, Yuma Ito, and Taisei Tanabe.
We partially obtained the research results from NICT's commissioned research No. JPJ012368C03101.
