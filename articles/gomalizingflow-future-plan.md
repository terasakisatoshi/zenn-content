---
title: "GomalizingFlow.jlの開発の経緯に関して"
emoji: "🌾"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Julia", "azarashi"]
published: true
---

# 本日は

[計算物理 春の学校 ２０２３](https://hohno0223.github.io/comp_phys_spring_school2023/) に参加された皆さんお疲れ様でした. ここでは [GomalizingFlow.jl](https://github.com/AtelierArith/GomalizingFlow.jl) に対する開発に至った経緯を書きます. 

https://github.com/AtelierArith/GomalizingFlow.jl

以下は完全にポエム(＝技術的な事柄を述べていない文章)です. Zenn では `type` として `tech`, `idea` という棲み分けをしています. この記事は `idea` という形で publish をしています.

# GomalizingFlow.jl ができるまで

GomalizingFlow.jl は格子上の場の理論における flow-based サンプリングを用いた配位生成アルゴリズムを提供するパッケージです．
プログラミング言語 Julia で書かれたものです. 実装のベースになったのは下記のツイートで紹介されている論文です:

https://twitter.com/MLPhysJP/status/1554395703429906433?s=20

[Introduction to Normalizing Flows for Lattice Field Theory](https://arxiv.org/abs/2101.08176) も参考にしました．

2年前ごろにこういうのをJuliaでできないっすか？と相談されたのがきっかけです. Python/PyTorch に関するコードは読めるし
PyTorchのレイヤーを Flux.jl に移植するのは以前 [Gomah.jl](https://github.com/terasakisatoshi/Gomah.jl) やったことがあるので容易でした.

https://github.com/terasakisatoshi/Gomah.jl

厳密に言えばこのリポジトリは Chainer から Flux.jl へのレイヤーコンバータです. とはいえ PyTorch の思想と Chainer の思想は子と親みたいなものなので
PyTorch -> Flux.jl への移植は ResNet に使われるレイヤーであれば問題なくできました.

１. こんな感じでできそうですね(・ω・｀)．
2. 素晴らしい, これできるといいですね
3. お，おう(´・ω・｀) この数式の意味を教えてください
4. これはね．ゴニョゴニョ
5. 学習のコードも書けそうなのでモデルの学習もGPUで計算できそうですね(・ω・｀)
6. 3 次元の理論できますか？
7. 設定ファイルでコントロールすればいけますね(´・ω・)
8. 現在に至る

気づいたらほとんど仕上げてたわけです. でも英文の論文で名前が残せてよかったです.

# GomalizingFlow.jl ができてから

## プレプリント

GomalizingFlow.jl はソフトウェア論文としてプレプリントが出ています. 富谷さんとの共著です.

https://twitter.com/MLPhysJP/status/1560741857222467584?s=20

## ワークショップ

Affine Coupling Layer を構成する CNN などのネットワークを改良した組合せ畳み込みレイヤーを導入し d=2,3,4 の次元で実験できるようにできました.

https://twitter.com/MLPhysJP/status/1583304885243908096?s=20

## 計算物理春の学校2023 の講義資料にて紹介いただきました

https://twitter.com/MathSorcerer/status/1635538733486452738?s=20
あとは講師として来日された Daniel Hackett (MIT) 氏の Introduction to normalizing flows for lattice field theory
のスライドの文献にも取り上げていただきました．

2023/3/28 の時点で https://hohno0223.github.io/comp_phys_spring_school2023/ に行けば当時の講義資料などが閲覧できるようになっています.

## パッケージ名の由来

`GOMAfu-san's implementation of norma_lizing flow` です(真顔).

https://twitter.com/AkioTomiya/status/1560623282981707781?s=20

履歴を見るとわかるのですが, 元々は LFT.jl というパッケージ名で開発していた様子がわかります.
コードを公開するときのパッケージ名どうしますかね

1. コードを公開するときのパッケージ名どうしますかねぇ(´・ω・｀) LFT は主語が大きいし.
2. Flow に関するものは欲しいよね．
3. せやなー(´・ω・｀)
4. ノーマライジング(Normalizing)とかけてゴーマライジング(Gomalizing)でどうでしょう？(´・ω・｀)
5. まぁいいんじゃない？
6. え， いいんですか？ (・ω・｀)
7. (確認のために 5 と 6) を２回ほど繰り返す
8. ま，いっか Goma が入るしw (・ω・｀)

### ウケたらしい(・ω・｀)

https://twitter.com/TomiyaAkio/status/1560683314503004160?s=20

https://twitter.com/AkioTomiya/status/1560691633334562816?s=20

各方面で宣伝ありがとうございます(´・ω・｀)

名前付けはパッケージ作成者の特権なので研究者の皆さんも自分の中での世界観を作って
Juliaで書いたパッケージを作っていくと良いと思います．

# 今後の開発に関して

私自身で自分で触っておかしな部分に気づいたらバグ修正程度のメンテはします. 過度な期待しないでください. もしあなたが, この分野に興味を持ち, 十分な人的，計算資源を持っているのであればコードを解析し，リポジトリのフォークを行い独自の機能を発展していくことを期待します.

GomalizingFlow.jl を管理している GitHub 組織は私個人が運営しているものであり, 国からの公的な支援を得ているわけではありません. 世の中のためを思い flow を学習するためのハードウェア, クラウド環境も含めた計算機, 費用も自費で賄ってきました. ところが昨今の社会情勢を受けて私個人の社会的な立場, 電気代, 半導体の高騰など情勢が大きく変化しました. そのため農家の支援や土地に肥料を与えない限り新しいタネを植えて畑を耕し作物を得るのは難しいと考えています.
作物は海外から輸入することはできます. ただ，それらが必ずしも美味しいとは限らないし，輸入がストップする可能性は十分にあります. 
健全な国力の発展には国内での自給率を上げることが欠かせないと考えています.
