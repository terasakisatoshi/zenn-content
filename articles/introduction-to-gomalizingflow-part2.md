---
title: "GomalizingFlow.jl 入門(Part2)"
emoji: "🦭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Julia", "azarashi"]
published: true
---

# 本日は

[計算物理 春の学校 ２０２３](https://hohno0223.github.io/comp_phys_spring_school2023/) に参加された皆さんお疲れ様でした. ここでは [GomalizingFlow.jl](https://github.com/AtelierArith/GomalizingFlow.jl) 

を実際に動かす方法を書きます. GomalizingFlow.jl は下記のようなことをするパッケージ...

![](https://storage.googleapis.com/zenn-user-upload/c983257fd588-20230403.jpeg)


ではなく格子上の場の理論, 特にスカラー場 $\phi^4$ 理論, における flow-based サンプリングを用いた配位生成アルゴリズムを提供するパッケージです. Part 1 は下記のリンクを参照ください.

https://zenn.dev/terasakisatoshi/articles/introduction-to-gomalizingflow-part1

ここではソフトウェアとして動かす方法を紹介します. GomalizingFlow.jl はプログラミング言語 Julia を用いて記述されており Julia で記述された Flux.jl という機械学習ライブラリを用いています.

# 動作環境

格子サイズが $L=8$ の2次元の理論であれば CPU でも動かすことができます. macOS(Intel) または Ubuntu が動作する GPU(GTX 1080Ti または RTX 3060) マシンで動作を確認しています.

# 動かし方

## Dockerfile から環境を構築する

Docker が動作する環境であれば `make` コマンドを叩いてイメージをビルドしコンテナを起動すれば flow の学習を進めることができます.

```console
$ git clone https://github.com/AtelierArith/GomalizingFlow.jl
$ make
$ docker-compose run --rm julia julia begin_training.jl cfgs/example2d.toml
```

学習が終了したあと下記のコマンドで Jupyter 環境を立ち上げ [playground/notebook/julia/analysis_tool.md](https://github.com/AtelierArith/GomalizingFlow.jl/blob/00891873c02a32c2b37764e0a3e54f8f38b3a13b/playground/notebook/julia/analysis_tool.md?plain=1#L1) をノートブックとして自己相関の計算やグリーン関数などの計算を進めることができます.

```console
$ docker-compose up lab
... localhost:8888 に移動 する
```

## すでに Julia が動かせる場合

すでに Julia の環境があれば次のようにして進めることもできます.

```console
$ git clone https://github.com/AtelierArith/GomalizingFlow.jl
$ julia --project=@. -e 'using Pkg; Pkg.instantiate()' # これは初回のみで良い.
$ julia --project=@. begin_training.jl cfgs/example2d.toml
```

`Pkg.instantiate()` によって `Project.toml` に記述されている依存関係 `[deps]` に記載されているパッケージを導入します. Python での `pyproject.toml` に相当します. `Manifest.toml` が得られますがこれは poetry での lock ファイルに対応します.

`--project=@.` は現在のディレクトリまたは親のディレクトリにある Project.toml を探しその環境をアクティベートする役割を果たします. 事前に環境変数 `JULIA_PROJECT=@.` をつけておくと入力を省くことができます.

## Python 環境はあるけれど... Julia がない場合

例えば GCP が提供する VertexAI は(お金さえあれば) Python + CUDA の環境を用意することができます. Julia の環境がデフォルトで用意されていないのが残念ですが Managed Notebook では次のようにして Julia をインストールできます.

```console
$ pip3 install jill # jill というコマンドが使えるようになる
$ jill install # 対話環境で y を入力
$ julia --version 
1.8.5
```

これで `すでに Julia が動かせる場合` の議論に帰着させることができます.

経験上, P100 のインスタンスだと CUDA.jl 周りが調子悪いので T4 インスタンスを使うことを勧めます. CPU は vCPU が 4 のインスタンスで十分です. GPU のメモリが多い環境ほど格子のサイズ $L$ が大きい場合の理論の解析を可能とします.

## 起動時はしばらく待つ

Julia は JIT コンパイルで動作するプログラムです. プログラムにおいて初回に実行される関数のコンパイルは時間がかかります. 数分はかかるので気長に待ってください.

# 学習スクリプトの引数

## 設定ファイルの指定

基本的には

```console
$ julia --project=@. begin_training.jl <設定ファイル>
```

のような使い方をします. `begin_training.jl` が受け付ける引数 `cfgs/example2d.toml` は色々変えることができます. 指定した物理系に関するパラメータ，モデルの定義に関するハイパーパラメータに関しての flow を学習することができます. `cfgs/example2d_E1.toml`, ..., `cfgs/example2d_E5.toml` は実装の元になった論文 [M. S. Albergo, G. Kanwar, P. E. Shanahan, Flow-based generative models for Markov chain Monte Carlo in lattice field theory, Phys. Rev. D 100 (3) (2019) 034515. arXiv:1904.12072, doi:10.1103/PhysRevD. 100.034515.](https://journals.aps.org/prd/abstract/10.1103/PhysRevD.100.034515) にある TABLE 1 の `E1`, ..., `E5` と対応しています.

もし 2 次元の理論に物足りない場合は `Nd=2` を `Nd=3` を変えれば形式的に 3 次元のための flow を学習することができます.

## デバイス番号の指定

`--device=<GPU デバイス番号>` でその番号に対する GPU で学習を進めることができます. 例えば 1 つのマシンに 2 枚のGPUが刺さっていたとします. `--device=1` とすることで `0, 1` とインデックスが付けられている `1` に対応するデバイス(GPU)を用いて学習を進めることができます.

Flux.jl を採用している GomalizingFlow.jl は今のところ複数 GPU で学習する機能を持っていません. Flux.jl の次世代版に当たる Lux.jl を使用すると複数 GPU での学習ができるようになる"はず"です.

# 評価指標監視ツール

学習時に epoch が回るたびにモデルの評価を行います. メトロポリス・ヘイスティング法によるサンプルを行なった際の受容確率や有効サンプルサイズ(ESS, Effective Sample Size) をログとして出します. これらの値は高くなるほど良いです(各々の最大値は 100, 1.0). 値の監視は UnicodePlots.jl を用いたスクリプトでコンソール上で確認することができます.

```console
$ julia --project=@. watch.jl <設定ファイル> --item acceptance_rate
```

`<設定ファイル>` は `begin_training.jl` に渡したものと同一のパスを要請します. `begin_training.jl` を動かすターミナルとは別のものを開いて実行してください.

# 学習済 flow による物理量の計算

学習済モデル(flow) が与えられればサンプル(配位)を生成しそれから定まる物理量を計算することができます. 相関関数やグリーン関数と呼ばれる量や Ising energy $E$, two-point susceptibility $\chi_2$ などの計算例は [この Pluto ノートブック](https://htmlview.glitch.me/?https://gist.github.com/terasakisatoshi/a834ece9474f9d0e72ec0ffd142df8a3) に記載しています. 2023/4/3 現在の実装で `cfgs/example2d_E1.toml` から `cfgs/example2d_E4.toml` のモデルから得られた数値計算の結果です. また $L$ の値を増やしても自己相関長を抑えられていることも確認できます.
これらの量の計算のスクリプトは [playground/notebook/julia/analysis_tool.md](https://github.com/AtelierArith/GomalizingFlow.jl/blob/00891873c02a32c2b37764e0a3e54f8f38b3a13b/playground/notebook/julia/analysis_tool.md?plain=1#L1) として提供されています.

# 学習コスト

一辺の格子のサイズ $L$ の値が増えていくと学習に必要な計算時間(≒ 評価指標として使われている受容率, `acceptance_rate` が 50 % となるまでに必要な epoch 数)が増えていきます. $L=12$ までは 1 日かければできますが, $L=14$ は数日を要します. flow-based によるサンプリングアルゴリズムを用いることで自己相関長に関する「難しさ」を回避できましたが, その回避が flow の学習の難しさに押し付けてられているように見えます.


# まとめ

- GomalizingFlow.jl の使い方を書きました.
- 計算結果も共有しました.
