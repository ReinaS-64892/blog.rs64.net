+++
date = '2026-10-20T17:38:59+0900'
title = '私がLinuxにVR環境を移してからのパッチのこと'
+++

## はじめに

このブログは Reina_Sakiria が FlugelKranz を作り Linux 環境に VR の環境までも移行し、 LinuxVR をしていくに当たって AI を使って行った パッチの数々の話の書き残し。

[参照](../freer-free-flight/)

## VRChat の AFK ができない

HMD を外しても、 AFK ができません ... ! 困る !!!

わざわざアバターの AFK には[気を使っている](../reina-sakiria-s-avatar-modification-in-around-2025#afkのセットアップ)タイプなので、出来ないとだいぶ不便なのよね

原因を調査したら、最初に見つかった問題は xrizer が AFK を伝える部分が未実装でした。

パッチを AI で当てさせました : https://github.com/ReinaS-64892/xrizer/tree/OpenXR-user-presence-proximity-input-(AI-generated)

動くね ... やったね！

issue : https://github.com/Supreeeme/xrizer/issues/434

## 初回の AFK ができない

これは WiVRn に問題がありました。

WiVRn は HMD 側の XR_EXT_user_presence を転送していましたが、初期値の送信がうまく行ってない様子なのか、初回の HMD 取り外しだと反応しないという問題がありました。

AI でパッチを作った後、 issue に報告したら WiVRn の方で修正がもうはいりました ... やった!

ありがとう WiVRn の人!

issue : https://github.com/WiVRn/WiVRn/issues/1155

## ValveIndex の手首がかなり吹き飛ぶ

なぜか、 VRChat でアバターの手首だけがかなり吹き飛ぶ問題が起きているんですよね。

これ、どこの層に問題があるのか (たとえば WiVRn 、 monado、 xrizer ) の特定にかなり苦労しまして ... 最終的には WiVRn のライトハウス系のトラッキングに対する予測時間リミッターの適用忘れが ValveIndex の skeletal hand root の部分にだけ合って、あんまり人々が踏まないような場所に抜けがあったせいでかなり調査に苦労しました。

VRChat 外で再現させるにあたって、`skeletal hand root` の特定というのは本当に大変だった ... AI あっても試行錯誤には人間が必要なせいでマジ大変だった ...

いまは AI にパッチを当てさせ、私の手元では吹き飛ばなくなっています。

issue : https://github.com/WiVRn/WiVRn/issues/1160

## ValveIndex の長時間での予測誤差が大きい

これは 一つ上の問題を調べている時に見つけた物を深堀した感じ。

そもそも、予測誤差に対してリミッターをかけることは、本質的な改善であるかというと微妙じゃないですか ...

なので、なんか途中で明らかなミスを見つけたりもしたので、AI に「もっと調べろ」と調べさせた結果改善させることが出来た。

その見つけたものの1つ目は、単位の変換ミス。けど、これ解決しなかったんよね、誤差レベルだった。

だけどこっちは、 upstream に修正がはいったんだよね ... ! 

MR : https://gitlab.freedesktop.org/monado/monado/-/merge_requests/3025

ありがとう monado の contributor の人 ... ！

んで2つ目、私自身完全に理解しているわけじゃないんだけど ...

monado の元の実装では、valve index のいくつかの位置(例えばレーザーの出す位置とか skeletal hand root のような、コントローラー独自になるべきいくつかの位置) に対して、オフセットをかけた先で予測を行う実装がなされていて、それは一番予測に寄与する IMU の位置では行われてなかったみたい。

IMU の情報をオフセットの位置に合わせて補正をかける [MR](https://gitlab.freedesktop.org/monado/monado/-/merge_requests/2625) によって良くなっていたみたいだけど ... 

WiVRn のような 3倍か 2倍くらい 有線 HMD よりも長い予測が必要になる環境下では、予測誤差がかなり大きくなってしまうようで ...

そこで、 IMU の位置で予測して、予測結果に対してそれぞれの位置へのオフセットをかけることで、かなり予測誤差を抑えることが出来たんだよね！

まぁ ... だいたい AI がやってくれただけだけどね()

IMU の位置と行っているが厳密には `tracking reference position` だそう。それが IMU の位置であると、決められているわけではないが ValveIndex と ViveTracker (2018/3.0) はそうなってるっぽい？
確証はない。

これによってかなり WiVRn での予測が良くなったので、私は嬉しいんだが ... upstream にもこの修正がちゃんとした実装で入ると嬉しいね。

(一応これによって、 予測時間リミッター を 50ms ぐらいまで大きくしても、かなり安定して使えるので、概ね追従性が上がったと行ってもいいかもね。解決自体は 予測時間リミッター　を強めにかけることでも解決はしていたから。)

issue : https://gitlab.freedesktop.org/monado/monado/-/work_items/627

## MotocalGUI

[参照](../motocal-gui/)

https://github.com/ReinaS-64892/MotocalGUI

## ライトハウスの BaseStation 規準での位置固定

MotocalGUI で空間合わせ周りの体験は良くなったんだが ... ライトハウスドライバーの出力する座標というのは時間経過でたまに若干ずれます。

というかこういった空間合わせの問題は、往々にして SteamVR でもあり、人々はこれに対していくつかの解を示しているようです。

### 既存手法

- [OpenVR-SpaceCalibrator bd_'s fork](https://github.com/bdunderscore/OpenVR-SpaceCalibrator) : Continuous Calibration

HMD にトラッカーを固定して、継続的にキャリブレーションし続けることで、たまに発生するそのズレを吸収するのだそう。

HMD とトラッカーの相対位置がずれないことを信用し、動いてなくても追従させる機能もあるそう ... ?

(私はこれを使うと、動きの幅の大きさ故か頻繁に頭のトラッカーのトラッキングを飛ばしてしまい、かえって精度が落ちるため、試したけどやめた。そもそも頭にトラッカー乗せたくないし)

- [OpenVR-SpaceOverride](https://github.com/Nyabsi/OpenVR-SpaceOverride)

HMD のトラッキングをそもそも 別のトラッカーに載せ替えちゃえばいいよねって発想っぽい。

最初にオフセット計算してその後はもう固定でトラッカー側のトラッキング情報に書き換えてしまえば安定するよねって話のよう。

- [OpenVR-SpaceSync](https://github.com/shinyflvre/OpenVR-SpaceSync)

どうやら SpaceOverride とは発想が逆方向のようで、　HMD にトラッカーを乗せたうえで、ライトハウスの側を HMD に乗ったトラッカーの情報を元に動かすのだそう。

SpaceOverride が HMD を ライトハウス系の空間に入れるのだとしたら、 SpaceSync は ライトハウス系の物を HMD の空間に入れようとしているというふうに見える。

- [QuestLHSync](https://github.com/CreoleVR/QuestLHSync)

HMD のカメラでベースステーション(? ... レーザーを捉えているのかな ... 詳しくはわからぬ)を捉えてそれをそれを元にライトハウス空間とHMDの空間をうまく合わせるそう

(非常に面白いが ... root 化しないと話にならない厳しさ ... SteamFrame をお使いならば選択肢かも知れない。)

- [lighthouse-freeze-patch](https://github.com/mog422/lighthouse-freeze-patch)

... バイナリパッチとか正気か !？ やり方が豪快すぎる ... ()

ライトハウスの座標系が再推定によってズレる問題を、バイナリパッチを当て不要な再推定を抑えたりすることで安定化させることが出来るそう。

OpenVR-SpaceCalibrator (等) との併用は必須だが、`Continuous Calibration` などの継続的なサイドのキャリブレーションが不要になるっぽい ... ?

### 私が出したやり方

既存手法、 bd_'s fork の Continuous Calibration を除いて全部、WiVRn/monado じゃつかえないんですけども ... (motoc は今のところ機能性は OpenVR-SpaceCalibrator bd_'s fork 相当)

でももっと良くしたいよね!

私としては 頭にトラッカー乗せるのはかなり嫌(すぐ飛ばしちゃうし、頭重くなるの嫌だし)、でも quest の root 化や バイナリパッチはさらに嫌 ... (steam frame ほしいね)

それで私は、`lighthouse-freeze-patch` のやり方を少し参考にして、 規準となる BaseStation を決めてそれを基準に固定することで解決することにしました。

この方法であれば、バイナリパッチも不要で安定したライトハウスの座標情報が得られます。

調査した感じ、どうやら steamvr_lh のドライバーからの出力は、相対的な角度はかなり安定しているので、絶対座標をどれかの基準にしてやればいいのです。

これで結構長時間 VRChat を遊んでいても再度のキャリブレーションをする必要がないくらいになって、私はかなり嬉しい。

ref : https://github.com/ReinaS-64892/WiVRn/tree/lh-stable-position

ただ ... これは、どこに提出したものか ... monado に上げるにしては ... ん〜すごく適当なためもっとやるべき実装が必要になりそうだが WiVRn ユーザーにとっては無意味な追加作業、それに規模も結構大きくなりそう ... (困った)

まぁ仕方がないので私の github においておくことにすることに ...

## WiVRn でのトラッカーのホットプラグ (動的追加&動的削除)

WiVRn では、ライトハウスのトラッカーが使えますが、起動時に繋がっていたトラッカーしか反応しないという問題があります。

ところで、それに関しては先行実装があるようで ... https://github.com/Eidenz/Monado

この問題は upstream となる monado にも issue / discussion はあるそう https://gitlab.freedesktop.org/monado/monado/-/work_items/605

とはいえ、時間がかかりそうな話なので、 Eidenz 氏の fork を hotplug の部分だけ WiVRn に移植させたらいい感じに動きました。やったね！

ref : https://github.com/ReinaS-64892/WiVRn/tree/hotplug

でも、これだけじゃ VRChat には動的に追加がされなかったんだよね〜

## xrizer のトラッカーホットプラグの対応

WiVRn の層がどうにかなったので次は VRChat と WiVRn/monado の間に挟まる xrizer の問題です。

xrizer が後から追加されたトラッカーを OpenVR のアプリに見せていなかったので AI にやらせたら解決してくれました。

ref : https://github.com/ReinaS-64892/xrizer/tree/fix/tracker-hotplug

ちなみに、WIVRn/monado 側の実装に不足がありトラッカーの電源オフが正しく出力されてなかったからそっちにパッチを当てたりし、 xrizer にも正しく正しく電源オフなどの通知を行わせることによって、やっとこさ、 WiVRn + xrizer + VRChat 環境下で、トラッカーを動的に入れ替えることが出来るようになったとさ ... 大変だった

(まぁほぼ AI がやっただけだから私たいしたことしてないけどね)

## おわり

出来たら全部が upstream に行ってくれれば嬉しいんだが .. まぁそうもいかないし、 issue はすぐに見られるわけでもないので気長に。

私のパッチ環境を使いたいのであるならば以下に集約してあるから、欲しい人は見て頑張ってセルフビルドしてみてね。(あなたの環境で絶対に動く保証も、正しく動くかどうかも保証はないけどね)

xrizer : https://github.com/ReinaS-64892/xrizer/tree/rs-main

WiVRn : https://github.com/ReinaS-64892/WiVRn/tree/rs-main
