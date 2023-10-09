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
Service Function Chaining (SFC) enhances network functionality by enabling various use cases, such as security services, NAT, DPI, and remote video production.
In SRv6, SFC is realized through nodes providing service segments linked to network service functions such as SFC Proxies.

> Service Function Chaining (SFC) {{rfc7665}} はネットワークの機能を向上させ、security services, NAT などの様々なユースケースを実現できる技術である。
> SRv6では、SFC は 各ノードが network service function に紐づいた SFCプロキシを行い、service segment を提供することで実現される {{!I-D.draft-ietf-spring-sr-service-programming}}。

However, conventional SRv6 SFC with an SFC proxy face issues related to geographical constraints and reduced forwarding efficiency.

> キャリアやコンテンツ事業者のネットワークでは、SLA など背景とした QoS サービスが提供される。
> しかし、QoS の最適化や利用量に追従したnetwork sercice function のスケーリングを、既存のSRv6 SFC では SR domain 上の任意の位置に network service function を展開することができない。これは、D-Plane/C-Planeそれぞれの制約による。

> * D-Plane: 従来の SRv6 SFC architecture では、SFC Proxy と SR-unaware な network service function と付随する 非SRv6 network を用意する必要があるため、SRv6 domain の any node で network service function を提供することが困難である。
> * C-Plane: network service function を自由な位置で提供する仕組みと、それを組み合わせてService Function Chainを構成するための仕組みが存在しない。

This document introduces the concept of "In-network SFC" as a solution to the conventional SRv6 SFC issue.
In the context of In-network SFC, network service functions can be strategically located at any point based on user demand, enabling SFC along the shortest path.

> このドキュメントでは、従来の SRv6 SFC の課題を解決する、"In-network SFC" を提案する。
> In-network SFC では、network service function は SR domain 上のあらゆる位置に配置可能とし、minimizing latency and maximizing bandwidth と組み合わせた optimal path や ユーザの需要に基づいたサービス提供などを実現可能にする。

To achieve In-network SFC, the components of the D-Plane and C-Plane are detailed as follows:

* D-Plane:
   * Specifies the details of "End.AN" as per {{!I-D.draft-ietf-spring-sr-service-programming}} and organizes design considerations for scenarios involving Anycast and Fast ReRoute.
   * Outlines the aspects related to treating each SRv6 Segment Endpoint Node as an "SRv6 Service Function Node".
* C-Plane:
   * Activates network service functions at the SR Segment Endpoint (Enabling End.AN).
   * Establishes service function chains at the SR Source Node (Adding SR Policy).
   * Classifies the target flow at the SR source node (Adding an encapsulation policy).

> In-network SFC を実現するために、D-Plane/C-Plane を下記のように整理する。
>
> * D-Plane
>    * 各中継ノードを "SRv6 Service Function Node" と定義し、Service Segment を提供する
>    * Anycast としての利用や Fast ReRouteについてなど、SRv6 Service Function Node の運用に関する機能の拡張
> * C-Plane
>    * network function を SRv6 Segment Function Node 上で有効化する
>    * SR source Node に service function chain を 発行する
>    * SR source Node で 分類した target flow に、対応する service function chain を encapulation可能にする

--- middle

# Introduction

Segment Routing over IPv6 (SRv6) {{!RFC8986}} enables packet steering through a set of instructions called the segment list by attaching an SR header to IPv6 packets.
Each SRv6 Segment Endpoint Node can provide various SRv6 Endpoint Behaviors, such as Node/Adjacency Segments, VPNs, and SFC {{!RFC7665}}.
This allows network operators to achieve advanced programmability for packets.

> SRv6 は、segment list と呼ばれる 命令セット を IPv6 packet の SR header に付与することで packet steering を行う技術である。
> それぞれの SRv6 Segment Endpoint Node は、様々な Srv6 Endpoint behaviors、例えば Node/Adjacency segment や VPN、SFC などを提供する。
> この仕組みにより、ネットワークオペレータは、パケット転送に対する先進的なプログラマビリティを実現することができる。

SFC enhances network functionality by providing various use cases, including security services such as FW, IPS/IDS, NAT services, and at the L7 level, features like DPI and remote video production through packet payload editing.

> SFC は、ネットワークの機能を向上させ、様々なユースケースを提供することができる。ユースケースとしては、including security services such as FW, IPS/IDS, NAT services、また、L7としては DPIや、ペイロード編集を通じたremote video production などが存在する。

In the current SRv6 architecture, SFC Proxy {{!I-D.draft-ietf-spring-sr-service-programming}} is used for SFC, but it faces two key issues:

* Geographic Constraints: It is incapable of deploying network service functions in desired or specific locations.
* Decreased Forwarding Efficiency: The service function chain cannot be established along the shortest path, resulting in reduced forwarding efficiency.

現在の SRv6 アーキテクチャでは、SFC のために SFC Proxy を利用している。
SFC Proxyは、SRv6 に対応していない従来の network service function を SR domain に対して提供するための技術である。
SFC Proxy は必ず SR domain の端点に配置され、SR domain 外のノードに転送されるという、地理的な制約がある。
この制約を取り払うことで、network service function を遅延や帯域などのメトリックに基づき最適な位置に配置して、転送効率を向上させたり、利用者の需要に応じた箇所でnetwork service function を展開するなどの柔軟なfunction提供が実現されると期待する。

End.AN is an SRv6 Endpoint behavior targeting at an SR-aware network service function, and enabling a node to process SRv6 natively and apply network service functions.
This allows for the provision of network service functions at geographically optimal locations, achieving SFC on the best path.
It also enables the utilization of available resources and the possibility for routers themselves to provide network service functions.

End.AN は、SR-aware な network service function を対象とする SRv6 Endpoint behavior であり、あるノードで SRv6 ネイティブにnetwork service function の提供を可能とする。
End.AN は、network function を SR domain 上の任意の箇所に配置することを可能とする。そのため、Service Function Chain の最適化や、ユーザの需要に応じた network service function の提供、需要に応じたService Segmentのスケーリングなどが可能となる。

This document describes a framework for In-network SFC using SRv6-based methods, covering both the data plane and control plane.
The generic SFC architecture is covered in {{!RFC7665}}, while the SFC architecture based on Segment Routing is covered in {{!I-D.draft-li-spring-sr-sfc-control-plane-framework}}, and therefore outside the scope of this document.

このドキュメントでは、SRv6 を用いて In-network に SFC を実現するための手法を、D-Plane/C-Plane双方の側面から説明する。
一般的な SFC architectureについては RFC7665で、その SFC architecture を Segment Routing に適用するための手法は draft-li で扱われているため、本ドキュメントの対象外とする。


# Terminology
This document leverages the terminology proposed in {{!RFC8402}}, {{!RFC8660}}, {{!RFC8754}}, {{!RFC8986}} and {{!RFC9256}}. It also introduces the following new terms.

* SRv6 Service Function Node: An SRv6 Segment Endpoint Node that provides SRv6 native network service functions as a service segment.
* In-network SFC: A technology that achieves SFC at any point of SRv6 domain by deploying SR-aware network service functions on any SRv6 Service Function Node.
* In-network SFC Controller: A controller that simultaneously manages SRv6 Service Function Nodes and SRv6 SR Source Nodes, overseeing the entire SFC within the SRv6 domain.

# Overview of SRv6 In-Network SFC Architecture
In the following sections, the SRv6 In-network SFC Architecture design is detailed.
It begins with the definition of design principles and is followed by the explanation of components and key technologies in both the data plane and control plane.

以降の章では、SRv6 In-network SFC Architecture のデザインを説明する。
まずはデザイン要件の定義をし、次にD-Plane/C-Plane 双方のコンポーネントやキーとなる技術を説明する。

## SRv6 In-Network SFC Architecture
{{!RFC7665}} outlines a procedure in which each packet is classified by the Service Classification Function, then forwarded to the Service Function Forwarder, and subsequently delivered to a specific network service function.
In the In-Network SFC architecture, the SRv6 SR Source Node classifies the flow and forwards it to specific SRv6 Service Function Nodes by specifying a Segment List that represents a particular Service Function Chain.

RFC7665 では、SFC の手続きを説明している。各パケットはService Classification Function で分類され、Service Function Forwarderに転送され、その後特定の network functionsに届けられることで、network service functions が適用され、SFC が実現される。
In-network SFC architecture では、SRv6 SR Source Node が パケットを識別し、特定の Service Function Chainを示すSegment listによって指定された Service Function Nodes に転送されることで、SFC を実現する。

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

Figure 1 shows an example of SRv6 In-network SFC with two network service functions.
Firstly, the SRv6 SR Source Node classifies the flow and encapsulates it with an SRH containing the segment list <s1, s2>.
Next, the SRv6 Service Function Node for s1 receives the packet and applies End.AN.
Finally, the SRv6 Service Function Node for s2 receives the packet and also applies End.AN, thus achieving SFC.

図一は2つのnetwork service functionsを用いた SRv6 In-network SFCの例を示している。
まず、srv6 sr source node が flow を識別し、service function chain を示す segment list <s1, s2> を付与したsrhでencapsulationし、転送を行う。
次に、s1 の srv6 service function node がパケットを受信し、end.an を適用する。
最後に。s2 の srv6 service function node がパケットを受信し、end.an を適用することで sfc が達成される。

Deploying multiple instances of the same network service function in an SRv6 domain enables the implementation of Service Function Chaining as a multipath.

ある SRv6 domain の中に 同一のservice function chain を複数デプロイすることで、Service Function Chain をマルチパスに利用できる。
SFCをマルチパスに構成する場合の注意点やD-Planeの仕様は4.1.2章で解説する。

## Architecture Principles
SRv6 In-network SFC is based on several key architectural principles:

1. In-network Processing: Forwarding packets along any specified (maybe shortest, maybe geographically distributed and so on) path.
2. On demand deploy functions : Deployment of Network Service Functions to Any Nodes on demand will be enabled
3 Integration with Traffic Engineering (TE): Representing SFC policy and QoS policy simultaneously by single SR Policy. Also additional control plane will be needed.
4. Per-Flow Identification: Classifying at the SRv6 SR Source Node based on the 5-tuples or even finer (for example, payload inspection) granularity.
5. Centralized Management: Managing Service Segments and per-flow Service Function Chains by a controller.
6. Ease of Deployment by enhancing legacy protocols: Employs existing protocol already widely used in controllers, such as BGP and PCEP.

SRv6 In-network SFC は、下記のアーキテクチャ要件に基づく:
1. In-network な処理: SFC を特定の経路（shortest path, 地理的に分散された経路など）で提供可能にする
2. 需要に応じたnetwork service function のデプロイ: 需要に応じた任意のノードでのネットワークションの提供を可能にする
3. TE との組み合わせ: SFC policy と QoS policy を同時に表現する SR Policy
4. per-flow での識別: フロー毎に異なる SFC Policy を付与するため、SRv6 SR Source Node において 5-tuple、あるいはそれ以上に細かい単位を識別
5. 一元管理: SFC に関するサービスのコントローラによる一元管理
6. デプロイメントを容易にする：PCEPやBGPなど、既存のコントローラで使われているプロトコルを採用する。

# Data Plane
The Data Plane is designed as follows to satisfy Architecture Principles 1, 3, and 4:

* End.AN-based Service Segment Provisioning: To achieve in-network processing with SRv6, the data plane utilizes End.AN to handle SR-aware network service functions.
* SRv6 Policy: Achieving SFC and QoS requirements through the Segment List.
* Per-Flow Encapsulation Policy: Applying per-flow requirements through the Encapsulation Policy using PBR.


## End.AN-based Service Segment Provisioning

By utilizing End.AN at an SRv6 Segment Endpoint Node, End.AN can be realized for provide Service Segments natively in SRv6. Simultaneously, this enables the provisioning of network service functions at any location based on customer demand within the SR domain.

Functions with the same role MAY be assigned as the same Service Segment within the SR domain.
By using Anycast SIDs, multiple nodes can be grouped as part of the same Service Segment.

End.AN MAY have optional arguments that can be passed as parameters to bound network service functions.

ある SRv6 Segment Endpoint Node において、End.AN を介して SR-aware network services を提供することで、Service Segment を in-network に実現する。
それと同時に、SR domain上の任意の箇所で Service Segment を提供可能となるため、顧客の需要に応じた箇所での network service function の提供が可能となる。

同じ役割を持つFunctionは、SR domain 上では同一の Service Segmentとして扱っても良い。（MAY）
Anycast SID を利用することで、複数のNodeを同じ Service Segmentのグループとして扱うことができる。

End.ANは、紐づく network service function に対して独自のパラメータを伝えるため、任意の引数を持っても良い。（MAY）


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
{: #end-an-pseudocode title="SID processing for SR-aware network service function (native)"}

### Anycast Segment
The concept of the Anycast Segment is introduced in {{!RFC8402}}. It is permissible to configure the same network service function segment as the same Anycast segment.
In such cases, the state between network service functions MUST be shared mutually.

Anycast Segment は {{!RFC8402}} で提案されている。
同一の network service function segment は 同一の anycast segment として設定しても良い。その際、FWやNATなどのサービスでは、network service function 同士のState は、互いに共有する必要がある。（MUST）

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
The Control Plane is designed as follows to satisfy Architecture Principles 2, 3, 4, 5, and 6:

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
NICT で実験しました
