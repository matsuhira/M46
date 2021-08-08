# M46について

## M46とは

IPv4アドレスを、IPv6アドレスにマッピングしたM46アドレス(M46A)を用いて、IPv4 over IPv6カプセル化、および、IPv4 - IPv6変換を行う技術です。<br>
IPv4プライベートアドレスに対応するために、M46Aは、IPv4プライベートアドレスを識別するためのplane-IDを含みます。
IPv6 Prefixを64bitとすると、Plane-IDには32bit割り当て可能ですので、2^32 = 約42億のプライベートネットワークの収容を可能にします。<br>
以下が、M46Aのアドレスフォーマットです。<br>

````
 |      96 - m bits      |          m bits          |     32 bits  |
 +-----------------------+--------------------------+--------------+
 |      M46A prefix      |  IPv4 network plane ID   | IPv4 address |
 +-----------------------+--------------------------+--------------+
````

IPアドレスは、アドレス全体で示されるホストアドレスと、それらを集約した経路を表す2つの意味がありますが、
ホストアドレスは、IPv4では/32、M46Aでは/128で表現され、
IPv4で/24で表現される経路は、M46Aでは、/120で表現されます。<br>

なお、M46は、Multiple IPv4 - IPv6 mappingの略、M46Aは、Multiple IPv4 - IPv6 mapped IPv6 addressの略です。<br>


## M46Eとは

M46Aを用いて、IPv4 over IPv6カプセル化に適用した技術です。IPv4パケットにIPv6ヘッダを付与するカプセル化を行いますが、その際、IPv6アドレスとしてM46Aを用います。<br>
IPv4の通信を、IPv6にマッピングしたIPv6アドレスを用いることにより、IPv6の空間での通信を可能にします。
これにより、バックボーンネットワークをIPv6 onlyにすることが可能になるほか、IPv6 onlyのネットワークを介したIPv4の通信を可能にします。<br>

M46Eは、M46Eの経路を広告するか否かで、M46Aの生成方法が異なります。経路広告する場合はM46E-FP、経路広告しない場合はM46E-PRと呼びます。<br>

なお、M46Eは、Multiple IPv4 - IPv6 address mapping encapsulationの略です。

### M46E-FP

M46E-FPは、M46AのIPv6 prefixに、M46A専用のprefixを割り当てるものです。<br>
M46Aのprefixを固定にすることにより、IPv4ネットワークが属するplane-IDと、IPv4サブネットが決まれば、M46Aアドレスが決まります。
すなわち、M46E-FPで必要となる設定は、M46A prefixと自身のplane-IDとIPv4サブネットの情報のみです。
通信相手のM46Aは宛先IPv4アドレスから自動的に生成できます。そして、通信相手への経路情報は、ルーティングプロトコルで伝えます。
ルーティングプロトコルは任意で、OSPFv3でも、IS-ISでも、BGPでも利用可能です。現実的かはともかく、スタティックルーティングでも動作する筈です。

設定は少なく済むメリットがありますが、IPv4の経路をIPv6アドレスにマッピングして経路広告する必要がありますので、総経路数は変わりません。<br>

なお、M46E-FPは、Multiple IPv4 - IPv6 address mapping encapsulation - fixed prefixの略です。

### M46E-PR

M46E-PRは、IPv6の経路広告をしない前提であり、IPv4サブネットが接続されるIPv6ネットワークのIPv6 prefixをテーブルに保持しておくことにより、
宛先IPv4アドレスが接続されるIPv6 prefixを解決してM46Aを求めるものです。IPv6が利用可能であることが前提で、IPv6通信を実現する経路広告に、
IPv4通信を乗せてやるというイメージです。<br>

M46Aの経路広告を行いませんので、エンドユーザ主導で実現できますが、IPv4サブネットと、そのサブネットが接続されるIPv6アドレスのprefixの対応関係を
テーブルで管理する必要があります。ただ、この対応関係は、そのplaneの利用者で同一の情報になりますので、設定簡易化の作りこみは可能と思います。<br>

M46E-PRには、ふたつのモードがあり、IPv4ホストとして動作するホストモードとIPv4ルータとして動作するルータモードがあります。<br>

ホストモードは、スタブネットワークが1サブネットであることを想定したモードで、ホストはNeighbour DiscoveryのNeighbour Advertisement、IPv4で言えばarpの解決により、
パケットを受信します。M46E-PRによるIPv4の通信は、IPv4ホストに対応するM46Aのアドレス解決が図られた際、M46E-PRは代理応答を行い、該当するM46A宛てのIPv6パケットを
受信します。受信後は、IPv6ヘッダを外し、デカプセル化し、IPv4パケットを取り出し、IPv4ホストに送信します。

なお、M46E=PRは、Multiple IPv4 - IPv6 address mapping encapsulation - prefix resolutionの略です。

### M46E-PT

M46E-PTは、M46E-FPで構成されたネットワークと、M46E-PRで構成されたネットワーク間を相互接続する技術で、M46Aの変換を行います。
変換されるのはprefixに該当する部分のみで、plane-idとIPv4アドレスは変換されません。<br>

なお、M46E-PTは、Multiple IPv4 - IPv6 address mapping encapsulation - prefix translatorの略です。

### トンネルとの違い

カプセル化を行う良く知られている技術はトンネルです。トンネルは、仮想的なpoint to point linkであり、L2のリンクを作る技術と言えます。<br>
一方、M46Eは、IPv4アドレスをIPv6アドレス空間にマッピングする技術で、ホストアドレスはもちろんのこと、サブネットもマッピング可能です。
すなわち、IPv4の通信を、IPv6アドレス空間で行うもので、IPv4で行われる通信を、IPv6アドレスを用いて行うものです。<br>

トンネルとM46Eには、カプセル化を行う点では同様ですが、いくつかの違いがあります。<br>

例えば、経路制御ですが、トンネルの場合、トンネルは実質的にはデータリンクになりますので、データリンク上でルーティングプロトコルを動作させる必要が生じます。
これに対し、M46Eでは、マッピングしたアドレスに対する経路制御は必要になりますが、こちらはアンダーレイで動作させれば良く、オーバレイで動作させる必要はありません。<br>

また、トンネルの場合、トンネルエンドポイントは一か所固定にせざるを得ず、冗長構成を取ることが簡単ではありません。
これに対し、M46Eでは、カプセル化／デカプセル化のポイントを複数構成し、例えば経路制御でポイントを選択するような運用も可能にできます。<br>

また、トンネルの場合、再帰的なトンネルを構成することも可能となりますが、M46Eの場合は、再帰的なカプセル化が行われることはありません。
本来は、再帰的なトンネルは望ましくなく、トラブルの原因となりかねないと思いますので、その点、トラブルの可能性を減らせていると思います。<br>



## M46Tとは

M46Tは、M46Aを用いたIPv4 - IPv6ヘッダ変換技術です。<br>
IPv4 - IPv6ヘッダ変換技術といえば、NAT64+DNS64が有名ですが、M46Tは、M46Aのアドレスマッピングルールを前提にしたことが大きな違いです。
M46T考案に至っては、IPv4サーバを、IPv6クライアントからアクセスする形態に割り切って限定しました。
それにより、プライベートアドレス割り当てのIPv4サーバへのIPv6によるアクセスを可能にした点と、IPv6アドレス直打ち、すなわち、DNS連携なしに
アクセス可能にする特徴があります。<br>

M46Tは、M46Eと組み合わせた利用も可能です。<br>

なお、M46Tは、Multiple IPv4 - IPv6 address mapping translatorの略です。

## M4P6Eについて

M4P6Eは、IPv4アドレスの共用に対応したIPv4 over IPv6カプセル化技術で、こちらは、IPv4アドレスに加え、ポート番号もIPv6アドレスにマッピングした技術です。
考え方は、M46Eと同様ですが、ポート番号も含めることにより、一個のIPv4アドレスを複数のホストでの共用を可能にします。<br>


## SA46Tについて

M46は、以前、SA46Tと呼んでおり、ほとんどの提案や活動は、SA46Tという名称を用いていた頃に盛んに行われました。<br>

SA46Tは、Stateless automatic IPv4 over IPv6 tunnelingの略で、技術的な特徴を素直に表現してみた名称を付けたのですが、
お恥ずかしいことに、tunnelingの名称を付けてしまいました。SA46Tという名称を用いた活動をいろいろ行っていたため、
SA46Tという名称をご存知の方もいらしたと思いましたが、技術の本質は、アドレスマッピングであり、トンネルではないことから、
思い切って名称を変えました。以下が新旧名称の対応関係を示します。<br>

| new | old |
----|----
| M46A | SA46T |
| M46E-FP | SA46T |
| M46E-PR | SA46T-PR |
| M46E-PT | SA46T-PT |
| M46T | SA46T-AT |
| M4P6E | SA46T-AS |


最初の提案は、SA46Tのみでしたが、その後、アイデアが広がりました。全てのアイデアを整理して名称を付け直した方が分かりやすくなるのではという動機もありました。
名称変更の結果、混乱させてしまっているかもしれませんが、このような背景と経緯がありました。<br>
