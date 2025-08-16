---
title: "IPv6 Mostly NW について学ぶ"
description: >-
  IPv6 Mostly NW について学ぶ。ただそれだけ。
author: taichi-e
date: 2025-08-17 12:00:00 +0900
media_subpath: /assets/img/20250817-ipv6MostlyNW/
categories: [LLM, emacs]
tags: [setup]
published: false
---

# IPv6 Mostly NW
## はじめに

インターネットは当初IPv4を基盤に拡大してきたが、IPv4アドレスはすでに枯渇しており、新たな持続的発展にはIPv6の導入が不可欠である。IPv6は128ビットアドレスを採用し、膨大なアドレス空間を提供することで、今後数十年にわたるインターネットの成長を支える基盤となる。これにより、従来NATに依存していたアドレス管理の課題が解消され、エンドツーエンド通信の回復も期待できる。

しかし現実の運用環境では、IPv4との互換性を無視できないため、当面はIPv6とIPv4の共存が前提となっている。Dual Stack方式は移行期における標準的な解として広く使われてきたが、ネットワーク機器や運用管理におけるコスト増大が問題となっている。この背景が、より効率的な移行モデルの必要性を浮き彫りにしている。

そこで登場したのが「IPv6 mostly NW」という概念である。これはIPv6を基盤としつつ、IPv4を最小限に利用する設計思想であり、既存インフラの制約を考慮しつつIPv6移行を加速するアプローチである。本稿ではその概要と意義について整理する。

### 背景

現在の多くのネットワークはDual Stackを採用しているが、IPv4アドレス枯渇の進行に伴い、IPv6を主体とした新しい運用形態への転換が求められている。IPv6 mostly NWは、IPv6をデフォルトに据えることで、インフラコストを削減しつつ効率的にIPv6化を推進できる利点がある。

さらに、アプリケーションやサービスがIPv6 readyになるにつれ、IPv4の役割は限定的になりつつある。特にクラウドサービスやモバイル通信では、IPv6 only環境がすでに広く実用化されており、その成功事例がIPv6 mostly NW導入の後押しとなっている。

一方で、IPv4接続性を完全に排除することは現実的ではない。そのため、NAT64/DNS64や464XLATといった補助的なトランスレーション技術を組み合わせることで、ユーザ体験を損なわずにIPv6中心の運用を実現する動機が強まっている。

### 動機

本稿の目的は、IPv6 mostly NWの概念を整理し、その特徴や適用範囲を明確にすることである。加えて、従来の移行方式であるDS-Liteなどとの比較を行い、相互補完の可能性を議論する。

また、IPv6移行技術の標準化動向を踏まえ、ISPや企業ネットワークが今後どのように対応すべきかを検討する。これにより、IPv6を主体とした運用に向けた具体的な方向性を示すことが可能になる。

最終的に、IPv6 mostly NWは移行期の一過性の技術ではなく、IPv6主導のインターネットを現実的に実現するための持続的な運用モデルであることを明らかにすることを目指す。

### 目的

本稿の目的は、IPv6 mostly NWの概念を整理し、その特徴や適用範囲を明確にすることである。加えて、従来の移行方式であるDS-Liteなどとの比較を行い、相互補完の可能性を検討する。

また、IPv6移行技術の標準化動向を踏まえ、ISPや企業ネットワークが今後どのように対応すべきかを検討する。これにより、IPv6を主体とした運用に向けた具体的な方向性を示すことが可能になる。

最終的に、IPv6 mostly NWは移行期の一過性の技術ではなく、IPv6主導のインターネットを現実的に実現するための持続的な運用モデルであることを明らかにすることを目指す。

### 手法

本稿では、IETFにおける提案や議論を中心に情報を整理する。具体的には、[draft-ietf-v6ops-6mops-03]におけるIPv6 mostlyの概念整理を出発点とし、関連するRFC群（RFC6877, RFC8781, RFC8925など）を参照する。

次に、IPv6 mostly NWと既存の移行方式であるDS-Liteの比較を行うことで、その位置付けと適用条件を検討する。これにより、IPv6主体のネットワーク運用に向けた現実的なアプローチを提示できる。

最後に、今後求められる技術的対応や運用上の課題を整理し、IPv6 only化への道筋を展望する。

## IPv6 Mostly NWとは

IPv6 mostly NWとは、通信の大部分をIPv6で行い、IPv4を必要最小限の補助技術として利用する設計思想である。このモデルでは、アプリケーションや端末がIPv6で動作することを前提に、IPv4通信は翻訳技術（例: NAT64, 464XLAT）を経由して提供される。

このアプローチは、IPv4枯渇に対応しつつ、運用コストを削減することを狙いとしている。Dual Stackのように両方のスタックを完全に維持する必要がないため、シンプルで持続可能なネットワーク運用が可能となる。

結果として、IPv6 mostly NWはIPv6への移行を現実的かつ加速的に進めるための戦略的選択肢として注目されている。

```
ソリューション概要 (Solution Overview)

IPv6-mostlyネットワークは基本的にDual Stackに似ているが、以下の要素が追加される。

	1. NAT64 ([RFC6146]) を提供し、IPv6-onlyクライアントがIPv4-only宛先に通信できるようにする。

	2. DHCPv4サーバが [RFC8925] で定義されたDHCPv4 Option 108を提供する。

端末がIPv6-mostlyネットワークに接続すると、その機能に応じてスタックを構成する。

	- IPv4-only端末 → DHCPv4でIPv4アドレスを取得

	- Dual-Stack端末 → IPv6を構成し、さらにDHCPv4でIPv4アドレスも取得

	- IPv6-only端末 → IPv6を構成し、DHCPv4でOption 108を要求して受け取った場合、IPv4アドレスを取得せずIPv6-onlyで動作

このモデルでは、IPv4-only、Dual-Stack、IPv6-onlyの端末が同一セグメントで共存できる。IPv6-only端末はNAT64を介してIPv4-only宛先にアクセスする。
```
[IPv6-Mostly Networks: Deployment and Operations Considerations][draft-ietf-v6ops-6mops-01] の日本語訳

## IPv6 DS-Lite との関係性

DS-Liteは、IPv6ネットワーク上でIPv4サービスを提供する技術であり、IPv6 mostly NWの運用モデルと密接に関係する。DS-Liteでは、IPv4パケットをIPv6トンネルで転送し、ISP側でCGNによりIPv4通信を実現する。

両者は「IPv6主体でIPv4を補助的に扱う」という共通の思想を持つが、実装方式に違いがある。IPv6 mostly NWでは、より幅広くNAT64や464XLATなどの技術を組み合わせ、柔軟にIPv4接続性を提供する点が特徴である。

したがって、DS-LiteはIPv6 mostly NWを支える一要素技術であり、他の移行技術と共存しながらIPv6化を推進する役割を果たす。

## 今後求められる対応

IPv6 mostly NWの普及に向けては、いくつかの課題が残されている。第一に、アプリケーションやミドルボックスがIPv6を前提とした設計に完全対応することが求められる。特にIPv4 literalに依存した実装は早急に解消する必要がある。

第二に、NAT64や464XLATといったトランスレーション技術の運用安定性を高めることが重要である。これには、[RFC8781]や[ietf-vpops-prefer]で議論されるPREF64発見手法の普及が欠かせない。

第三に、運用者にとっての管理手法の標準化とベストプラクティスの共有が必要となる。これにより、IPv6 onlyネットワークの実現に向けた移行期の混乱を最小化できる。

## 用語(Terminology)

- Endpoint (エンドポイント): ネットワークに接続され、オペレータからホストと見なされるデバイス。ただし、場合によっては他のシステムへ接続を拡張しルータの役割を担うこともある。例として企業のノートPCやテザリングを有効化したスマートフォンがある。

- Network segment (ネットワークセグメント): VLANやブロードキャストドメインのように、ホストが同一IPサブネットを共有するリンク。

- PREF64 (NAT64プレフィックス): IPv6クライアントからIPv4サーバへの通信に用いられるIPv6プレフィックス [RFC6146]。

## 参考文献

[draft-ietf-v6ops-6mops-01]: https://datatracker.ietf.org/doc/draft-ietf-v6ops-6mops/
- [IPv6-Mostly Networks: Deployment and Operations Considerations][draft-ietf-v6ops-6mops-01]

[draft-ietf-v6ops-6mops-03]: https://www.ietf.org/archive/id/draft-link-v6ops-claton-03.txt
- [IPv6-Mostly Networks03][draft-ietf-v6ops-6mops-03]

[rfc8925]: https://datatracker.ietf.org/doc/html/rfc8925
- [IPv6-Only Preferred Option for DHCPv4][rfc8925]

[rfc8781]: https://datatracker.ietf.org/doc/html/rfc8781
- [Discovering PREF64 in Router Advertisements][rfc8925]

[rfc6877]: https://datatracker.ietf.org/doc/html/rfc6877
- [464XLAT: Combination of Stateful and Stateless Translation][rfc6877]

[ietf-vpops-prefer]: https://datatracker.ietf.org/doc/html/draft-ietf-v6ops-prefer8781-01
- [Prefer use of RFC8781 for Discovery of IPv6 Prefix Used for IPv6 Address Synthesis][ietf-vpops-prefer]

[XLATs]: https://datatracker.ietf.org/doc/html/draft-equinox-v6ops-icmpext-xlat-v6only-source-01
- [Using Dummy IPv4 Address and Node Identification Extensions for IP/ICMP translators (XLATs)][XLATs]
