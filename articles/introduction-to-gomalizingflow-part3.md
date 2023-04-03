---
title: "GomalizingFlow.jl 入門(Part3)"
emoji: "💠"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Julia", "azarashi"]
published: false
---

# 本日は

[計算物理 春の学校 ２０２３](https://hohno0223.github.io/comp_phys_spring_school2023/) に参加された皆さんお疲れ様でした. ここでは [GomalizingFlow.jl](https://github.com/AtelierArith/GomalizingFlow.jl) 
のパッケージをベースに 4 次元の理論を動かすことを考えます. そこで得たアイデアは affine coupling layer を構成するネットワークアーキテクチャーを変更することでした. 学習時の受容率の向上スピードを上げることができました.

```julia
(x, y, z, t, inC, B) # select 2 axes , say, "x" and "y" from ["x", "y", "z", "t"] in this example
->
(x, y, inC, z, t, B) # permutedims
->
(x, y, inC, (z, t, B)) # treat (z, t, B) as a batch axis.
->
(x, y, inC, (z * t * B)) # reshape
->
(x, y, outC, (z * t * B)) # apply 2D convolution
->
(x, y, outC, z, t, B) # reshape 4D -> 6D
->
(x, y, z, t, outC, B) # permutedims to restore the array data
```

