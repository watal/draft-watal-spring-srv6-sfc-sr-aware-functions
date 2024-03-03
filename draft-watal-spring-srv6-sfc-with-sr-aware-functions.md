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
* Simplicity: no SFC proxies, so that reduces nodes and address resource consumption.

--- middle

# Introduction
Segment Routing over IPv6 (SRv6) {{!RFC8986}} enables packet steering through a set of instructions called a segment list.
Each SR segment endpoint node provides SRv6 Endpoint Behaviors, including Prefix/Adjacency segments, VPNs, and Binding Segments.

Service Function Chaining (SFC) {{!RFC7665}} can be used in various scenarios (e.g. FW, IPS, IDS, NAT, and DPI).
The SFC based on Segment Routing (SR) is defined in {{!I-D.draft-ietf-spring-sr-service-programming}}, which describes SFC proxies like End.AS/AD/AM are necessary to use SR-unaware functions.

This document describes an architecture for SRv6 SFC with SR-aware functions, which provides comprehensive management of SRv6 network resources and services.

# Terminology

## Terminology Defined in Related RFCs and Internet-Drafts
The following terms are used in this document as defined in the related RFCs and Internet-Drafts:

* SR, SR Domain, Segment ID (SID), SRv6, SR Policy, Prefix segment, Adjacency segment, Anycast segment, Active segment, and distributed/centralized/hybrid control plane defined in {{!RFC8402}}.
* SR source node, transit node, and SR segment endpoint node defined in {{!RFC8754}}.
* SRv6 SID function and SRv6 Endpoint behavior defined in {{!RFC8986}}.
* SFC, SFC proxy, and service classification function defined in {{!RFC7665}}.
* service segment, SR-aware service, SR-unaware Service, End.AS, End.AD and End.AM defined in {{!I-D.draft-ietf-spring-sr-service-programming}}.
* Headend, Color, and Endpoint defined in {{!RFC9256}}.
* Quality of Service (QoS), Service Level Agreement (SLA), and Service Level Objective (SLO) defined in {{!RFC9522}}.
* forwarding plane (FP), control plane (CP), management plane (MP), application plane (AP), northbound interface, southbound interface defined in {{!RFC7426}}.
* Path Computation Client (PCC), Path Computation Element (PCE), and Traffic Engineering Database (TED) defined in {{!RFC5440}}.
* BGP Flow Specification defined in {{!RFC8955}}

## Newly Defined Terminology
The following terms are used in this document as defined below:

* SRv6 Service Function Node: an SR segment endpoint node that provides SR-aware functions as service segments.
* Classification Rule Controller: applies sets of SR policy and flows to SR Source Nodes.
* Service Function Controller: applies service segments to SRv6 Service Function Nodes.
* SRv6 Controller: controls SRv6 services comprehensively, consisting of a Service Function Controller, a PCE, and a Classification Rule Controller.
* SRv6 Managers: manage SRv6 SFC infrastructure, consisting of a Virtualized Network Function (VNF) Manager, a Virtualized Infrastructure Manager (VIM), and a data collector of network metrics.

## Requirements Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all capitals, as shown here.

# Design Objectives and Assumptions
## Goals/Objectives
SRv6 SFC Architecture is designed with two main objectives:

* Comprehensive management: a centralized controller for SFC, handling SR Policy, link-state, and network metrics.
  When providing SRv6 services, meeting SLAs for each customer is required.
  These SLAs consist of one or more SLOs such as availability, latency, and bandwidth.
  In an SRv6 SFC network, service segment provisioning, link-state collection, and SR policy calculation are required to meet SLOs, respectively.

  {{!RFC8402}} outlines a hybrid control plane that merges distributed control plane and centralized control plane.
  In this hybrid control plane, forwarding information like Node/Adjacency SIDs are advertised mutually by distributed SR nodes via IGPs such as ISIS and OSPF, while other information like SR Policies and service segments are reconciled by a centralized controller.

  Software-Defined Networking (SDN) {{!RFC7426}} provides centralized management of the network by a controller and a manager.
  That centralized management reduces operational costs through abstraction and automation.
  レイヤのabstractionは、
  C-Plane による操作は、ポリシの計算をautomationさせる
  このアーキテクチャでは、SRv6 を SDN framework に対応させ、包括的な管理を実現する。

  SDN Framework を採用するもう一つの理由は programmability である。
  SRv6 Controller provide programmability.
  XXX: can build SFCs, apply them to specific flows, set SLOs as an intent, and Anycastやプロテクションの説明
  XXX: AP を用意することで、オペレータに対して APIを提供できる。

* Simplicity: no SFC proxies, so that reduces nodes and address resource consumption.
  Network complexity increases operating costs.
  Generally, using a variety of protocols in a network raises operational costs, including designing, building, monitoring, and troubleshooting.

  Using SFC proxy, forwarding overhead may increase due to additional header manipulations.
  End.AN を利用することでSFC Proxy をなくす

  SRv6 has various functions such as VPN, QoS, redundancy, and disaster recovery.
XXX: functionとserviceをごっちゃにしている．再整理が必要．
XXX: disaster recoveryとredundancyの違いもよくわからない．
  By using SR-aware functions, all forwarding instructions, including instructions to each network function, can be expressed as a simple set and applied to each flow.
XXX: instructions, including instructionsが気持ち悪い.
XXX: instructions to each network functionsの意味がわからない．
XXX: all forwarding instructions can be expressed as a simple set.が意味がわからない. それはSRv6の基本．"SR-aware functionsを使うことによって"の説明になっていない
XXX: 上記について，can be expressedというのは表すことも可能だし表さないことも可能という意味だが，SRv6でinstructionをsetで表さない場合なんてない．
XXX: all forwarding instructions can be applied to each flowもよくわからない．当たり前じゃん？適用できないinstructionとかなんの意味があんの

## Assumptions
To achieve these objectives, this architecture is based on two main requirements:

* Straightforward extension of the SRv6 Network Programming model

  The protocol used in this architecture MUST be compatible with SRv6.
  This simplifies the operation of services such as traffic steering including SFC, redundancy, and Fast Reroute (FRR).
XXX: redundancyとFast Rerouteの違いがわからない．半分包含関係にあると思う．
  This architecture uses standardized SRv6 protocols such as BGP, PCEP, IS-IS, OSPF, TI-LFA, and Anycast SID.
BGP / PCEPなどはSRv6プロトコルとは言わないと思う．

このアーキテクチャは SRv6 に準拠するため、SR-unaware function も対応可能にするが、objective を満たすためには SR-aware function を前提にする
  The controller manages not only SR-aware functions but also SR-unaware functions and other SRv6-TE services.

* SDN Framework compliance and comprehensive management of SRv6 SFC by controllers
  A controller MUST be used to provide a consistent policy.
XXX: MUSTが同様
  To simplify building and operating, the controller MUST use standardized protocols and abstracted service interfaces.
XXX: MUSTが同様
  This also provides programmability by controlling policies that meet a user's intent including SFC and quality of service (QoS).
XXX: これは利点であってRequirementsではない．
  This architecture uses controllers to manage service segments, SFCs, TEs, VPNs, link-state, and network metrics.
XXX: これもIntroductionで説明している話であってRequirementsではない．

# Overview of Architecture
Figure 1 illustrates overviews of this architecture.

~~~ drawing
 +------------------------------------------------+
 |               Application Plane                |
 +------------------------|-----------------------+
                          | Control Plane Northbound Interface
 +--- SRv6 Controller ----v-----------------------+
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
{: #overview title="Overview of SRv6 SFC Architecture with SR-aware Functions"}

This architecture is based on SDN {{!RFC7426}} separating the forwarding plane (FP), control plane (CP), management plane (MP), and application plane (AP).
Each plane has the following roles:

* forwarding plane: パケットの転送を
   * Provides SR-aware function using End.AN.
   * Classify flow and apply them to TE application with PBR.
   * Ensures redundancy with Anycast.
   * Ensure protection with FRR.
* control plane: forwarding plane を作る
   * Collects link-state including SRv6 locator, prefix, behavior, and delay.
   * Calculates and provisioning SR Policies.
   * Applies SR Policies to each flow by provisioning flow classification rules.
   * Manages the provisioning of Service Segments to SR-aware functions.
* management plane: ネットワークデバイスの monitoring, maintenance を行う
   * Monitors and deploys network functions.
   * Manages hypervisor resources.
   * Collects metrics of devices, network functions, and SFC services.
* application plane: ユーザに API を提供し、ネットワークにインテントを適用させる
   * Provide an interface to operators or customers.
   * Applying intents defined in {{!RFC9315}}, including Operational, Rule, Service, and Flow intents.

Each component communicates using standardized protocols.
These are designed to be loosely coupled and cooperate by using an abstraction layer.

This document suggests handling control plane by application plane, but a detailed design of application plane is out of the scope of this document.
This is because application plane components and abstraction layers should be designed based on individual network utilization and operator intent.
In the following sections, details of forwarding plane, control plane, and management plane are explained.

# Forwarding Plane
A forwarding plane is responsible for providing SFC through packet classification, SRv6 encapsulation, and forwarding.
In this architecture, all forwarding plane components are located within the SR domain.

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

Figure 2 shows an example of SFC with two network functions.
Firstly, the SRv6 SR source node classifies the flow and encapsulates it with an SRH containing the segment list <S1, S2>.
Next, the SRv6 Service Function Node (S1) receives the packet and applies a network function associated with an End.AN S1.
Finally, the SRv6 Service Function Node (S2) receives the packet and also applies a network function associated End.AN S2, thus achieving SFC.

## End.AN-based Service Segment Provisioning
End.AN provides an SR-aware function.

Functions with the same role MAY be assigned as the same service segment within the SR domain.
By using Anycast-SIDs, multiple nodes can be grouped as part of the same service segment.

End.AN MAY have optional arguments.
This can provide additional programmability by embedding network function instructions in the segment list.

任意のノードで提供可能になるよ
また、latency を減らすかも

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
A control plane is responsible for enabling comprehensive management of SRv6 SFC.
It enables SR-aware functions as service segments and specifies SR Policies including SFC for each flow.
control plane has a Northbound API to receive user requests and a Southbound API to manipulate forwarding plane.

~~~ drawing
 +---------------- SRv6 Controller ----------------+
 | +--------------+ +-------------+ +------------+ |
 | |Classification| |    Path     | |  Service   | |
 | |     Rule     | | Computation | |  Function  | |
 | |  Controller  | |Element (PCE)| | Controller | |
 | +------|-------+ +-^---------|-+ +------|-----+ |
 +--------|-----------|---------|----------|-------+
   Classification link-state SR Policy Enable/Disable
        Rule       (BGP-LS) (PCEP/BGP) a Service Segment
   (BGP Flowspec)     |         |     (End.AN SID:S1)
 +--------|-----------|---------|----------|----------------------+
 | +------v-----------|---------v-+ +------v--------------------+ |
 | |     SRv6 SR Source Node /    | |       SRv6 Service        | |
 | |    Service Classification    |-|         Function          | |
 | |           Function           | |           Node            | |
 | +------------------------------+ +---------------------------+ |
 +--------------------------- SR domain --------------------------+
~~~
{: #cp title="Control Plane"}

The SRv6 Controller consists of the following three components:

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

## Path Computation Element (PCE)
PCE is a controller that provides SR Policy.
As an Active Stateful PCE, it establishes sessions with all PEs in an SR domain and manages SFCs.
SR Policies MUST support both explicit and dynamic paths.
For dynamic path, Constrained Shortest Path First (CSPF) consider not only SFC but also QoS.

It acquires the Traffic Engineering Database (TED) of the SR domain using BGP-LS and deploys SR Policies via PCEP {{!RFC5440}} or BGP SR Policy {{!I-D.draft-ietf-idr-segment-routing-te-policy}}.

The SR Policy can utilize CSPF to meet various requirements, including SFC and QoS.
SR Policies can be defined on a per-flow or per-TE basis, providing flexibility.
XXX: この行が多分英語として崩壊している．can be defined on per-TE basisの意味がよくわからないのと，providing flexibilityが唐突に登場しているように感じる．
The BGP-LS service segment is needed to calculate dynamic paths considering service segments and states of the network functions.
XXX: BGP-LS service segmentが必要とされている，という言い方が謎．普通そのような言い回しはしないと思う．
XXX: pathがconsiderすることはない．

## Classification Rule Controller
A Classification Rule Controller specifies flows to apply specific SFC.
XXX: specifyは明示するとか詳細に述べるとかなので，ここでは不適当な気がする．

For communication with each node, an extended protocol based on BGP Flowspec is used for SR Policy.
SR Policy specification consists of three components: endpoint, color, and policy name.
XXX: SR Policy specificationという単語も唐突に登場しているように見える．しかもいわゆる今まで上で言っているSR Policyとの違いがわからない．

A set of endpoints and color is provided as described in {{!I-D.draft-ietf-idr-ts-flowspec-srv6-policy}}.
XXX: ここで言いたいこと整理する

# Management Plane
A management plane is responsible for configuring network function instances, monitoring resources, and collecting network metrics.
The details of each manager are outside the scope of this document, as the southbound interface of the management plane may be different for each service and hardware architecture.

~~~ drawing
 +----------------- SRv6 Manager ------------------+
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
 +------------------- SR domain -------------------+
~~~
{: #mp title="Management Plane"}

Figure 4 shows examples of managers that MAY be added to a management plane:
* VNF Manager: handles deployment and scaling of network functions.
* This manager considers redundancy and link utilization optimization.

* VIM: monitors hypervisor resources on SRv6 Service Function Node.
   * In SRv6 SFC, a hypervisor managed by a VIM MAY be located in virtualized spaces within routers or on generic servers.
* Network Metrics Manager: collects metrics for SRv6 policy calculation and evaluation.
   * Metrics are collected from multiple data sources, including IPFIX, TCP statistics, and SRv6 path tracing {{!I-D.draft-filsfils-spring-path-tracing}}.
   * Metrics can be used for PCE calculation parameters.

# Security Considerations
In this architecture, network functions are globally accessible via IPv6, since the network functions are SRv6 service segments.
If a network function has a security vulnerability, this node could be attacked, and other nodes in the SR domain could also be lateral movement attacks.
XXX: なんかこれは適当なことを言っているように感じる．
XXX: もし脆弱性があったら危ないし水平展開されますって当たり前の話だし，別にSRv6に限った話では全然ない．
XXX: どっちかっていうと外から好き勝手トラフィックエンジニアリングをさせることを防ぐ手段などは提供してないから気をつけてねとかそいういう感じだっともう．
XXX: SRv6 service segmentだからIPv6からアクセスできます自体ロジックが崩壊している気がする．
Therefore, by default, an information of each service segment MUST NOT be leaked outside of a domain, network operators MUST use filtering to drop packets from unauthorized sources to service segments.
XXX: ここのMUST NOTも適当に入れている感じがしてならない．ちゃんと考えられていないならMUST/MUST NOT/MAYとか入れないほうがいい．
XXX: そもそもleakするなとか当たり前の話すぎて，MUST NOTみたいなことを言うのはおかしいし，ばれなければ脆弱性あってOKみたいなのはセキュリティ業界が最も嫌うことの1つだと思う．
XXX: 後半についてはどこかの話のパクリなキモsるが，service segmentsへのパケットはdropしろは適当すぎる感じがする．そもそもどこでどのようにdropするべきなのかとか触れられていないし．

The security requirements and mechanisms described in {{!RFC8402}}, {{!RFC8754}}, and {{!RFC8986}} are also applicable to this document.

--- back

# Acknowledgments
{:numbered="false"}
The authors would like to acknowledge the review and inputs from Mitsuru Maruyama, Katsuhiro Sebayashi, Yuma Ito, and Taisei Tanabe.
We partially obtained the research results from NICT's commissioned research No. JPJ012368C03101.
