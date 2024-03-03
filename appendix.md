# Highly Reliable Firewall Service Using SRv6 End.AN
If you implement a firewall as an SR-aware function at an SRv6 End.AN node, you can forward packets using anycast SID and also achieve TI-LFA Fast Reroute {{!I-D.draft-ietf-rtgwg-segment-routing-ti-lfa}}.
This makes clustering firewalls easier as well.
XXX: ここ，アイデアとしては価値があると思っているけど，あまりに曖昧でちゃんと練れていないからどっかに移して今回は入れなくて良いと思う．

# SR-aware NAT
In the Interop Tokyo 2023 ShowNet's backbone SRv6 network, they had to decapsulate packets to conduct Network Address Translation.
XXX: to conduct NAT -> for SR-unaware NAT ?
If you use SR-aware NAT, you don't have to decapsulate the packets when traversing the NAT function.
This contributes to achieving a simpler network design.
XXX: これも結構良い例だとは思うけど，ちゃんと理解しないで適当なこと言ってる感があるから割愛してもいい気がする．.mdファイルを別に分けるとかして．

# Intent-based SFC management
{{!RFC9315}} defines intent as "operational guidance and information about the goals, purposes, and service instances that the network is to serve."
The architecture for providing SRv6 SFC with SR-aware functions is based on the SDN Framework {{!RFC7426}} and includes an application plane.

TODO: 追記

XXX: 2行目は何言ってるのか全然わからない．

# Flexible and Low-latency Remote Production Service
In the context of video remote production, you can perform video processing within an SRv6 network by combining multiple network functions (SFC).
If you have to distribute multiple connections from several sources, you can also use multicast packets in the SRv6 network.
XXX: 色々説明不足すぎる．やはりユーザdefinedなFunctionということでEnd.ANのAppendixにギリギリ入れるか入れないかとかな気がする．

