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

This document describes an architecture for SRv6 SFC with SR-aware functions, which provides comprehensive management of an SRv6 network resources and services.

# Terminology

## Terminology Defined in Related RFCs and Internet-Drafts
The following terms are used in this document as defined in the related RFCs and Internet-Drafts:

* SR, SR Domain, Segment ID (SID), SRv6, SR Policy, Prefix segment, Adjacency segment, Anycast segment, Active segment and distributed/centralized/hybrid control plane defined in {{!RFC8402}}.
* SR source node, transit node, and SR segment endpoint node defined in {{!RFC8754}}.
* SRv6 SID function and SRv6 Endpoint behavior defined in {{!RFC8986}}.
* SFC, SFC Proxy, and service classification function defined in {{!RFC7665}}.
* service segment, SR-aware service, SR-unaware Service, End.AS, End.AD and End.AM defined in {{!I-D.draft-ietf-spring-sr-service-programming}}.
* Headend, Color, and Endpoint defined in {{!RFC9256}}.
* egress node, ingress node, metric, Quality of Service (QoS), Service Level Agreement (SLA), and Service Level Objective (SLO) defined in {{!RFC9522}}.
* forwarding plane (FP), control plane (CP), management plane (MP), application plane (AP), Northbound Interface, Southbound Interface and Service Interface defined in {{!RFC7426}}.
* Path Computation Client (PCC), Path Computation Element (PCE), and Traffic Engineering Database (TED) defined in {{!RFC5440}}.

## Newly Defined Terminology
The following terms are used in this document as defined below:

* SRv6 Service Function Node: an SR segment endpoint node that provides SR-aware functions as service segments.
* Classification Rule Controller: applies sets of SR policy and flows to SR Source Nodes.
* Service Function Controller: applies service segments to SRv6 Service Function Nodes.
* SRv6 Controller: provides comprehensive management of SRv6 SFC, consisting of a Service Function Controller, a PCE, and a Classification Rule Controller.
* SRv6 Managers: manage SRv6 SFC infrastructure, consisting of a Virtualized Network Function (VNF) Manager, a Virtualized Infrastructure Manager (VIM), and a data collector of network metrics.

## Requirements Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all capitals, as shown here.

# Design Objectives and Requirements
## Goals/Objectives
SRv6 SFC Architecture is designed with two main objectives:

* Comprehensive management: a centralized controller for SFC, handling SR Policy, link-state, and network metrics.
  When providing SRv6 services, meeting SLAs for each customer is required.
  These SLAs consist of one or more SLOs such as availability, latency, and bandwidth.
  In an SRv6 SFC network, service segment provisioning, link-state collection, and SR policy calculation are required to meet SLOs, respectively.

  {{!RFC8402}} outlines a hybrid control plane that merges distributed control plane and centralized control plane.
  In this hybrid control plane, forwarding information like Node/Adjacency SIDs are advertised by distributed SR nodes via IGPs such as ISIS and OSPF, while other information like SR Policies and service segments are reconciled by a centralized controller.
XXX: advertisedも相互の話ではないので，日本語的には相互に交換される，みたいな単語を考える

  As an approach to achieving centralized control, {{!RFC7426}} defines Software-Defined Networking (SDN).

XXX: achive centralized controlという言い回しが，言いたいことはわかるし伝わると思うが，英語的に何かがしっくりこない．
XXX: achiveという言葉は，特定のある目標"値"を超える．すなわち達成するというニュアンスがある．なのでコントロールを達成するという言葉はおかしい．例えば，ある状態に到達することは一般的には実現する(realize)とかの方がまだしっくりくる．
XXX: この行はRFC7426はSDNという単語を定義していること以外言っていない．基本的には．なので，なぜこの前提的な話をしているのかが次の行でSDNで定義されているcentralized controlを直接的に説明する文章でないなら，意味がわからない浮いた行になる．
XXX: この行はSDNを実現するための手法(approach)の"1つとして"SDNが定義されている．みたいな文章になっているわけだが，ネットワーク界隈において，SDN以外のcentralized controlは存在しないと思うし，centralized controlすることをSDNと呼ぶ気がする．すなわち，ネットワーク界隈においてはSDN=networkのcentralized controlだと思っているので，アプローチの一つみたいなことを言われると違和感がある．あくまで一つなのであれば，ほかのアプローチについても説明を加えてほしい(SDN以外のcentralized approachについて)
  Centralized management of SRv6 SFC components reduces operational costs through abstraction and automation.
XXX: 上でcentralized controlという言葉が唐突に出てきているにも関わらず，その次の行でその説明をするどころか別の単語(centralized managemente)の話をし始めているところに違和感を感じる.
XXX: SFC componentsに関わらずSRv6 componentsはcentralizedにmanageされるんじゃないの？なんでわざわざSFCという単語で絞りを入れているのかよくわからん
XXX: abstraction and automationについて，"何の"が説明されていない．どうabstractしてどうautomateするのかがこの行か次の行くらいで説明されていないなら，説明不足

  Additionally, programmability can be provided by using SRv6 Controller and abstracted application planes.
XXX: can be provided by using がすごく冗長に感じる．例えば, Additionally, SRv6 Controller and abstracted application planes provide programmability.
XXX: abstracted application planeというのは抽象化されたapplication planeなので，application plane自体がそもそもcontrol planeのabstractionをするレイヤー(plane)なのに，さらにそれをabstractedしているとはどういうことやねん．ChatGPTの対話層でも挟んでるのか？みたいに思わせる.
XXX: 上のセクションは，Centralized managementについて説明している．それに加えて，説明しているにも関わらず，こっちはCentralized managementではなく, SRv6 Controller/abstracted application planesについて説明しているので，"Additionally"ではない．(Centralized management = SRv6 Controller and abstracted application planesでない限り)
XXX: Controllersと複数形で書いているが，Centralizedなんだったら単数系では？大量にSRv6 Controllerがあるわけじゃないよね？(内部的に幾つかのコンポーネント(サブコントローラ)があるのはわかるが, また冗長化したら増えるだろうとかはあるとは思うが．).

  Operators can build SFCs, apply them to specific flows, set SLOs as an intent, and determine fallback policies via controllers' API.
XXX: Operatorsができるのは当たり前じゃん？OperatorsはコントローラをAPIから操作できるという話と，そのコントローラができる話を分けるべきだと思う．
XXX: fallback policiesが唐突．できることを思いついた順に整理せずに記載していっている印象がある
XXX: 経路をフローに適用するのか，フローを経路に適用するのか．後者な気がする．


  For these reasons, this architecture is designed to provide comprehensive management of SRv6 SFC.
XXX: ここは意味わからない．例えば1セクション前を見ても，OperatorはAPIを通じてSFCs(経路)を適用できる，"だから"このアーキテクチャはcomprehensive managementを提供するためにデザインされている．は論理が崩壊している．目的と手法が逆．

* Simplicity: no SFC Proxies, so that reduces nodes and address resource consumption.
  Network complexity increases operating costs.
  Generally, using a variety of protocols in a network raises operational costs, including designing, building, monitoring, and troubleshooting.

  A complex forwarding plane can be a cause of increasing latency.
XXX: forwarding planeが複雑であることはすなわちレイテンシを増加させる可能性がある，はなんのために言ってるのかわからない．別に我々の提案はforwarding planeがシンプルになるわけではないと思う．(End.ANのレイテンシの削減はほとんど効果がないことが研究でわかっている)
  Using SFC proxy, forwarding overhead may increase due to additional header manipulations.
XXX: レイテンシーがとか言っている割に，End.ANを使うとどのように減るのかなど説明がないまま次の話に飛んでいる．SFC Proxyをなくすんですよ我々の提案は，という一番肝心な説明もないし．

  SRv6 has various functions such as VPN, QoS, redundancy, and disaster recovery.
XXX: functionとserviceをごっちゃにしている．再整理が必要．
XXX: disaster recoveryとredundancyの違いもよくわからない．
  By using SR-aware functions, all forwarding instructions, including instructions to each network function, can be expressed as a simple set and applied to each flow.
XXX: instructions, including instructionsが気持ち悪い.
XXX: instructions to each network functionsの意味がわからない．
XXX: all forwarding instructions can be expressed as a simple set.が意味がわからない. それはSRv6の基本．"SR-aware functionsを使うことによって"の説明になっていない
XXX: 上記について，can be expressedというのは表すことも可能だし表さないことも可能という意味だが，SRv6でinstructionをsetで表さない場合なんてない．
XXX: all forwarding instructions can be applied to each flowもよくわからない．当たり前じゃん？適用できないinstructionとかなんの意味があんの

  For these reasons, this architecture is designed to minimize forwarding plane components and use SR-aware functions.
XXX: 意味がわからない．このセクションは，SR-aware functionsはsimpleなsetとして表すことができるから，このアーキテクチャはSR-aware functionsとforwarding plane componentsを最小化する"ために"設計されている．になるが，ロジックが崩壊していて意味不明

## Requirements
To achieve these objectives, several key requirements are as follows:
XXX: 英語として崩壊している．key requirements are as follows.なんてのはおかしい．重要な要求事項が以下，みたいな感じ

* Provide SFC using an SR-aware function
XXX: そもそも目的と手法がよく整理されていない上に，手法と要求事項もぐちゃぐちゃ．SR-aware functionsは1つの手法であって，要求事項ではない．

  An SR-aware function MUST be used to achieve simple SFC without proxies.
XXX: 勝手にSFC proxiesをproxiesと略すべきではない．ネットワークの世界のプロキシはいろいろなものがある．
XXX: 何がMUSTなのか意味がわからない．SRv6ネットワークを使うときにEnd.ANもEnd.AMもMUSTじゃない．
XXX: MUSTとか実装の要件みたいなのはInformational RFCで書くべき内容ではない．
XXX: 何かをachiveするためにはMUST，みたいなのも意味不明．MUSTの使い方について書いてあるRFCを一読すべき
  This minimizes the number of nodes, address resources, and protocols.
XXX: これは別の場所で説明しているし，そもそもこれは要求事項ではない．
  This architecture uses End.AN.
XXX: これは手法の説明であって，要求事項ではない．

* Straightforward extension of the SRv6 Network Programming model
XXX: そもそも，InformationalにRequirementsとかMUSTとかが入っている時点で意味がわからない．ベストプラクティスとかやってみた結果の知見とかをまとめたりするのがInformationalだという理解が正しいとすると，なんで要求事項とかが出てくるのかわからない．

  The protocol used in this architecture MUST be compatible with SRv6.
XXX: MUSTが同様
  This simplifies the operation of services such as traffic steering including SFC, redundancy, and Fast Reroute (FRR).
XXX: redundancyとFast Rerouteの違いがわからない．半分包含関係にあると思う．
  This architecture uses standardized SRv6 protocols such as BGP, PCEP, IS-IS, OSPF, TI-LFA, and Anycast SID.
BGP / PCEPなどはSRv6プロトコルとは言わないと思う．

* SDN Framework compliance and comprehensive management of SRv6 SFC by controllers

  A controller MUST be used to provide a consistent policy.
XXX: MUSTが同様
  To simplify building and operating, the controller MUST use standardized protocols and abstracted service interfaces.
XXX: MUSTが同様
  The controller manages not only SR-aware functions but also SR-unaware functions and other SRv6-TE services.
XXX: この行が唐突に出てきてよくわからない．SR-awareだけでなく"SR-unaware"もというのは当然なのに，なんでわざわざ強調しているのかわからない
  This also provides programmability by controlling policies that meet a user's intent including SFC and quality of service (QoS).
XXX: これは利点であってRequirementsではない．
  This architecture uses controllers to manage service segments, SFCs, TEs, VPNs, link-state, and network metrics.
XXX: これもIntroductionで説明している話であってRequirementsではない．

# Overview of Architecture
Figures 1 and 2 illustrate overviews of SFC with SR-aware and SR-unaware functions, respectively.
XXX: with SR-aware/SR-unawareって全部だし，SFCなんだから書く必要がないと思う．

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

This architecture is based on SDN {{!RFC7426}} separating the forwarding plane (FP), control plane (CP), management plane (MP), and application plane (AP).
Each plane has the following roles:

* forwarding plane: responsible for providing SR-aware functions, classifying services, and applying SFCs for each flow.
   * Provides SR-aware function using End.AN.
   * Conducts flow classification and TE application with PBR.
   * Ensures redundancy and protection with Anycast and FRR.
* control plane: responsible for controlling Service Segment, calculating SR Policy including SFC, and providing classification rules for each flow.
   * Collects link-state including SRv6 locator, prefix, behavior, and delay.
   * Calculates and provisioning SR Policies.
   * Applies SR Policies to each flow by provisioning flow classification rules.
   * Manages the provisioning of Service Segments to SR-aware functions.
* management plane: responsible for deploying SR-aware functions, managing resources, and collecting network metrics.
   * Monitors and deploys network functions.
   * Manages hypervisor resources.
   * Collects metrics of devices, network functions, and SFC services.
* application plane: responsible for providing application interfaces to specify user intent, topology visualization, and notification.
   * Provide an interface to operators or customers.
   * Applying intents defined in {{!RFC9315}}, including Operational, Rule, Service, and Flow intents.

Each component communicates using standardized protocols.
These are designed to be loosely coupled and cooperate by using an abstraction layer.

This document suggests handling control plane by application plane, but a detailed design of application plane is out of the scope of this document.
This is because application plane components and abstraction layers should be designed based on individual network utilization and operator intent.
In the following sections, details of forwarding plane, control plane, and management plane are explained.

# Forwarding Plane
forwarding plane is responsible for providing SFC through packet classification, SRv6 encapsulation, and forwarding.
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

Figure 3 shows an example of SFC with two network functions.
Firstly, the SRv6 SR source node classifies the flow and encapsulates it with an SRH containing the segment list <S1, S2>.
Next, the SRv6 Service Function Node (S1) receives the packet and applies a network function associated with an End.AN S1.
Finally, the SRv6 Service Function Node (S2) receives the packet and also applies a network function associated End.AN S2, thus achieving SFC.

## End.AN-based Service Segment Provisioning
End.AN provides an SR-aware function.

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
control plane is responsible for enabling comprehensive management of SRv6 SFC.
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
management plane is responsible for configuring network function instances, monitoring resources, and collecting network metrics.
management plane Southbound Interfaces are specific to individual services and hardware architectures, therefore, details on each manager are outside the scope of this document.

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

Figure 5 shows examples of managers that MAY be added to management plane:

* VNF Manager: handles deployment and scaling of network functions.
   * This manager MAY consider redundancy and link utilization optimization for scaling network functions.
* Virtualized Infrastructure Manager (VIM): monitors hypervisor resources on SRv6 Service Function Node.
   * In SRv6 SFC, a hypervisor managed by a VIM MAY be located in virtualized spaces within routers or on generic servers.
* Network Metrics Manager: collects metrics for SRv6 policy calculation and evaluation.
   * Metrics are collected from multiple data sources, including SRv6 path traces, IPFIX, and TCP statistics.
   * Metrics can be used as inputs for controllers described in this document.

# Security Considerations
In this architecture, network functions are globally accessible via IPv6, since the network functions are SRv6 service segments.
If a network function has a security vulnerability, this node could be attacked, and other nodes in the SR domain could also be lateral movement attacks.
Therefore, by default, an information of each service segment MUST NOT be leaked outside of a domain, network operators MUST use filtering to drop packets from unauthorized sources to service segments.

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
{{!RFC9315}} defines intent as "operational guidance and information about the goals, purposes, and service instances that the network is to serve."
The architecture for providing SRv6 SFC with SR-aware functions is based on the SDN Framework {{!RFC7426}} and includes an application plane.

# Acknowledgments
{:numbered="false"}
The authors would like to acknowledge the review and inputs from Mitsuru Maruyama, Katsuhiro Sebayashi, Yuma Ito, and Taisei Tanabe.
We partially obtained the research results from NICT's commissioned research No. JPJ012368C03101.
