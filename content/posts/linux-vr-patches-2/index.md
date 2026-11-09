+++
date = '2026-11-09T15:44:47+0900'
title = 'Linux VR環境でのパッチのこと - 2'
+++

## はじめに

このブログは [私が Linux に VR 環境を移してからのパッチのこと](../linux-vr-patches/) の続きにあたります。

## Steam Frame を手に入れました 

いろいろとあって ... Steam Frame を手に入れました ... 経緯は ... ん〜、ちょっとここで書くのはやめておきます。

当人に VRChat とかで会ったら直接聞いてね。

Steam Frame を手に入れた影響で、これまでの Quest2 での生活に終焉を迎え、コントローラーがまともなものになります ... !

## FlugelKranz の Steam Frame 対応

まずはこれだよね ... !

トリガー (上?) に Drag を割り当てて、 Y や D-pad up に 回転 Drag を割り当てて ... ってやるだけ

正直、 Valve Index の トラックパッドよりも使いやすくて、以前よりも、機動力がちょっと上がった気がする。 (入力に関しては事故がかなり減った)

https://github.com/ReinaS-64892/FlugelKranz/pull/5

## WiVRn だと Steam Frame の d-pad の一部が使えない

なんか ... WiVRn の実装ミスがあった、AI にここ原因あるっていってた、明確になんかただの記述ミスがありそうってなったから自分の手で書き換えて治ったから PR にしたら取り込まれた。

やったね！

https://github.com/WiVRn/WiVRn/pull/1195

## VRChat で DFR ! (VRS)

[dfr-for-vrchat](../dfr-for-vrchat/)

## VRChat で QV 式 DFR !

DXVK 層でするの ... いや ... かなり厳しいとは思うけど ... いけるんじゃないかな ... ?

を形にしてみました。 AI が

{{< x user="ReinaS_64892" id="2118265818220007691" >}}

これは ... 結構微妙で QV 式のトレードオフが相当フラグメントシェーダーが厳しい場所じゃなきゃ意味がなかったです。

パフォーマンスがぎりぎり下がることはないし、視覚的トレードオフを除けば GPU 使用率を下げれるメリットはありますが、コストパフォーマンス的な意味で VRS 式 DFR に勝てない。

だれか、超高解像度 HMD を Linux で動かせるようになったら試してみてね ... !

それと、現状出来ないけど、多分少し改造すれば、注視点をさらに高解像度にすることも出来なくはないと思う。私はやらなかったけど

https://github.com/ReinaS-64892/dxvk/tree/qv-dfr

## Stream Frame にて WiVRn の手の描画がおかしい

ずっとあったのでフレンドが直していたらしいのでそこの情報もらって調査したけど ... 結論は SteamVR のバグで、ワークアラウンドをする気にもならなかったから調査で終わった。

https://github.com/WiVRn/WiVRn/issues/1140

https://github.com/ValveSoftware/SteamVR-for-Linux/issues/975

## WiVRn にポート転送機能を追加してみた

[Babble Frame Runtime](https://github.com/Project-Babble/Babble-Frame-Runtime) を導入しまして ... でも、 WiVRn がどの PC につなぐか選んでいるところがあるので、探索して投げる先をどうにかしてくれる手段も会ったが ... そいうのは避けたいんですよね。

じゃあ WiVRn にポート転送機能を付けて、 [Babble Frame Runtime](https://github.com/Project-Babble/Babble-Frame-Runtime) のような、 Steam Frame 内で動いて、 PC とやり取りしたい奴らは、全部 Steam Frame のポートとおしゃべりしてもらって、 WiVRn が適宜転送すれば ... いいよね！

ってことで AI に雑にパッチを当てさせました。

普通に汎用仕様なので便利 

https://github.com/ReinaS-64892/WiVRn/tree/rs-port-forwarding

## QuestLHSync

Steam Frame は root を取らなくてもカメラにアクセスできます ... 便利〜！

ってなわけで、最近出てきた 空間合わせソリューションであるところの QuestLHSync を使いたくなったんですが、当然 WiVRn 対応は発展途上 ... (というか若干放置されてないか？)

そんなわけで AI に突かせたらいい感じになりました ... SteamVR で出来てたことを、ほぼ同レベルで出来るまでつなぎ合わせてくれました ... AI つよいね ()

後ついでに、 UI がバグってる問題とかも直させたため ... 結構差分が大きくなってしまった気がするが ... まぁいいんじゃないかな。

https://github.com/ReinaS-64892/WiVRn/tree/fix/questlhsync-tracking-quality

https://github.com/ReinaS-64892/QuestLHSync/tree/feat/wivrn-tracking-stability

https://github.com/ReinaS-64892/xrizer/tree/questlhsync-base-stations

なお issue の方はあんまり動きなさそう ... 

https://github.com/CreoleVR/QuestLHSync/issues/6

ちなみについでに、上で書いた WiVRn のポート転送機能を経由するように使っていて便利。

## おわり

便利に使うためには仕方ないんだけど ... パッチなしのオリジナル環境にもう戻れない ... ん〜

まぁ仕方がないね (Windows/SteamVR だったら諦めるしか、なかったことがどうにかなってるんだからね！)
