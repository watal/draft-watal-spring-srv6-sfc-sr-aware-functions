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

This document describes the architecture of SRv6 SFC with SRv6-aware functions, which enables comprehensive SFC management.

This architecture provides the following advantages:

* Simple design with all devices and protocols using SRv6.
* Low latency and reduced capital expenditure (CAPEX) through proxy-free SFC.
* Scalable control plane and reduced operating expenses (OPEX) via centralized management.
* Enhanced programmability through centralized control of TEs, including SFC and QoS, and the use of user-defined network functions.

XXX: In-networkというワードが特に新しいのでそこを推しているかのようなタイトルになっている．一番与えたい印象や新しいポイント嬉しいポイントが明確になったら再度タイトルについて検討する → 利点を箇条書きで示した
XXX: CAPEXみたいなマーケットに出た時にどうなるかわからない信憑性の薄いことを主要メリットとして挙げるのではなく，よりシンプルなアーキテクチャでビルディングブロックが減る，みたいな誰がみても明らかに真な理由を書く．
XXX: low latencyは我々が実験したところ，もちろん削減方向にはなるが，全く気にするに値しないレベルなので，主要なメリットの一番として挙げるには明らかに不適
XXX: 2行目は，CAPEXはやめて，本文のProxyが入らなくなるよね．ということは必要なネットワーク機器が減るので，すなわちこれはCAPEXの削減につながるかも(MAY)ね．くらいの言い方で"本文に"書く
XXX: through proxy-free SFCは, proxy-free SFCだと何でLow Latencyでreduce CAPEXできるのかが全く説明されていない．つまり論理の飛躍があって，聞き手を納得させられない．ちゃんと説明を書く．(CAPEXは消す), 最小限のコンポーネント数で，みたいな書き方をする．
XXX: reduced OPEXはなんかおかしい．lower OPEXの方がまとも．見たこともある．
XXX: via centralized managementがScalableなcontrol planeとOPEXの削減を実現する，というのは論理が飛躍している
XXX: 基本的に上記のように論理が飛躍しているので，scalableであるということとcentralizedというのは現時点では別項目にするべき．一緒にするにしてもscalable and centralized control planeみたいな言い方にするべき
XXX: through / via / by あたりを適当に使っている．ちゃんと使い分けるべし
XXX: QoSやuser-defined functionsはおまけで，End.ANだとSRの世界で完結してコントロールできて嬉しいことの方が推したいポイントなのに，余計なことが書かれている割に重要なことが書かれていないのでなおす

--- middle

# Introduction
Segment Routing over IPv6 (SRv6) {{!RFC8986}} enables packet steering through a set of instructions called a segment list.
Each SR segment endpoint node provides SRv6 Endpoint Behaviors, including Prefix/Adjacency-Segments, VPNs, and Binding Segments.

Service Function Chaining (SFC) {{!RFC7665}} can be used in various scenarios (e.g. FW, IPS/IDS, NAT, and DPI).
Within the current SRv6 architecture, SFC proxies like End.AS/AD/AM are necessary to apply network functions.
In addition, the SFC architecture based on Segment Routing is described in {{!I-D.draft-li-spring-sr-sfc-control-plane-framework}}.

XXX: Withinに違和感．他のいい言い回しがありそう
XXX: currentはRFCになることを考えるとおかしな表現．20年後のIETFで読まれて，Currentっていつだよって話に絶対になる．もっと具体的にかけ

This document describes the SRv6 native SFC architecture.
This architecture aims to enhance the capabilities of SFC by using SRv6-aware network functions.
XXX: aimは主語がすることなので，architectureがaimしているわけではなく, architectureは我々によってある目的のために作っているので, どっちかっていうとaimedとかの方が正しいはず
XXX: enhance the capabilities of SFCって具体的に何なのかさっぱりわからない．何のためのなのかもわからないし，capabilitiesをただ拡張しましょうみたいなのは認められない．ミニマムでビットと計算リソースを食わずに必要なことを実現するのが素晴らしいとされるコンピュータネットワークの世界で，ただいろんなことができるようになります！でfatなプロトコルにしていくみたいなのは許されんはず
XXX: theじゃない気がする．．

The SRv6 native SFC architecture provides the following benefits.

* Simple architecture designed for all devices and protocols to use SRv6.
   * TE Service including SFC and QoS guarantees, are completed within an SRv6 domain.
   * Using the SRv6 ecosystem technologies such as D-Plane (FRR/TI-LFA, Anycast, and Prefix Aggregation), C-Plane (IS-IS, BGP, PCEP), and M-Plane (Path Tracing, IPFIX, and OAMs).
* Low latency and reduced capital expenditure (CAPEX) through proxy-free SFC.
   * Efficient forwarding closed within the SRv6 domain.
   * Minimal resource design, eliminating SFC proxies and IP addresses of non-SR components.
* Scalable control plane and decreased operating expenses (OPEX) via centralized management.
   * Comprehensive control of SRv6 domains with an SDN approach.
   * Providing an efficient operation system through centralized management.
* Enhanced programmability through centralized control of TEs, including SFC and QoS, and the use of user-defined network functions.
   * Demand-driven provisioning of Service Segment and SFCs per flow.
   * Provide network functions as user-defined behavior with End.AN

XXX: C/D-Planeを含む以上にタイトルに対する付加情報がない．なので，HOGEFUGAする, PIYOのための, みたいな情報を付加するべき / coverという言葉はあまり使っている例はない．RFCを読んでいると，大抵describeとかdefinedとか → This Document defines から始まる文章で価値について説明．

XXX: genericという言い方が主観的というか，なんかニュアンスに違和感を感じる / and therefore outside the scope of this document.が英語の文として崩壊している → generic という表現は廃止し，この I-D のスコープの話は Terminology に移動

XXX: guaranteesがQoS知っている人には意味がないし，知らない人にもあまり意味がない気がするから冗長
XXX: are completed withinが直訳すぎる．SRv6 domainの中で完結するとは具体的にどういうことかの説明を入れる

XXX: the SRv6 ecosystem technologiesが冗長．the SRv6 ecosystemとthe SRv6 technologiesが指すものは変わらない．もっというと，SRv6とSRv6 technologiesが指すものも変わらない．なので単純にSRv6と言えば良い．
XXX: such asというのはfor example的な言葉なので，D-Plane, C-Plane, and M-Planeみたいに全部を列挙している場合に使う言葉ではない．
XXX: この文で言いたいことは，既存のSRv6の技術を使い回す(破壊しない)ということなのに，それについてどこにも書いていない．書く
XXX: 例えばC-Plane(IS-IS/BGP/PCEP)などは，SRv6を知っている人からすると当たり前の話で，しかも我々が改めて説明する必要はない(他のRFCで定義されてるんだからそれを引用すれば良いだけ)ので，消す

XXX: SRv6ドメインにclosedだからforwarding がEfficientというのも論理が飛躍している．
XXX: Abstでも書いたがLow Latencyを売りにしてはいけない(Lowじゃないから)
XXX: 直前に are completed within an SRv6ドメインとか書いてて，同じことを繰り返しいうのは良くない(しかも表現が微妙に変わっている)

XXX: Minimal resource  designとeliminating SFC proxiesは言っていることが一緒．
XXX: Minimal resource designという言葉がそもそも違和感がある. 最小資源設計...？なんぞその造語...という印象
XXX: eliminateも気持ち悪い．これは悪いものを取り除く的なニュアンスのワードで, そもそもdesignとか言ってるんだったら，取り除くとかではなく不要と書けばいいだけの話
XXX: non-SR componentsとかいう造語を作るの禁止．SR-unaware Functions的な使われている言葉で説明できるものはそれを踏襲する．造語，ダメ，絶対
XXX: eliminate IP addressなんて言い方はしない．節約とか削減とかそういう言い方

XXX: SDN approachって具体的にどういうこと？何が言いたいのか不明．RFCは，それに基づいて実装ができるようなちゃんとした仕様である必要があるので，こういう曖昧な表現は避けるべき．具体的に何か既存の手段を指し示したいのであればRFC番号とともに具体的に説明するべき
XXX: efficient operation systemを提供しているわけではない．あとoperation systemみたいな言い方はこの業界の人は絶対にOSを想起するので勘違いを生みかねない．throughというワードがしっくりこない(上記のby/via/throughの違いをちゃんと理解して書くべし)

XXX: Demand-driven provisioningというような感じで適当な造語をポコポコ爆誕させてはならない．
XXX: Demand-driven provisioningを提案(仕様)しているような書き振りだが，実際にはその素養を作っているだけで，具体的にどうやってDemand-Driven provisioningを実現するかを定義しているわけではないので，ここでこのアーキテクチャのメリットとして書くべきではない．Usecaseとして，Appendixに書くのがせいぜいだと思う．

To realize the SRv6 Native SFC, there are some requirements as follows:
XXX: Native SFC / SR-aware functions / native SRv6 functionなどの表記揺れをちゃんと整理してどれを使うべきか考えた上で直す
XXX: requirementsというより，勝手に設計しているのでそういう書き方にする．一般常識的に，要求事項，みたいなのを書くべきではなく，こういうことをするには，こうする必要があります，というだけ

* D-Plane: utilizes SRv6-aware network functions and ecosystems. XXX: utilizeが怪しい．ecosystemsも合わせて消すことになるかも
  * SRv6-aware network functions: using the "End.AN" behavior as described in {{!I-D.draft-skyline-spring-srv6-aware-services}}.
  * compatible with SRv6 ecosystems XXX: そもそも機能を足すだけなのだから，既存SRv6をぶっ壊さないことは自明なので，書かない．もしくはuSIDのドラフトを参考に，言い回しを修正する
* C-Plane: uses SRv6 Native SFC Controller to provide programmability to SRv6 network operators by establishing SFCs and manipulating SRv6 Service Function Nodes.
XXX: SRv6 Native SFC Controllerという造語を気軽に作ってはいけない
XXX: provide programmabilityするためにNative SFC Controllerを使うはロジックがおかしい．programmabilityがあるからコントローラを作ることができるわけなので．
XXX: SRv6に新しい機能とか足そうぜというドラフトを書いていて，SRv6 network operatorsを対象にしているかのような自明なことは書かなくて良い．
XXX: establishing SFCsすることによってSRv6 Native SFC Controller を使うわけではないのでロジックが崩壊している.
XXX: ここでwatal氏が言いたいことがadd/deleteなのであればprovisioningという言葉を使うべき．manipulateは細かく操作する，いじるみたいな言葉なのでそれ自体をデプロイするとか潰すという意味で使うのには不適切

  * Enabling End.AN: activates network service functions at SR segment endpoint nodes.
  * Adding SR Policy: provisions service function chains at SR source nodes.
  * Applying SR Policy per flow: classify the target flow and apply SR policy at SR source nodes.

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
## Objectives

SRv6 in-network SFC architecture is designed to enhance the capabilities of service provisioning with SFC.

This architecture for aims the following objectives:

* Reducing latency and CAPEX (CAPital EXpenditure).
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
 +---------------- SRv6 Native SFC Controller ----------------+
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
 +---------------- SRv6 Native SFC Controller ----------------+
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
