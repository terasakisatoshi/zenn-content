---
title: "GomalizingFlow.jl 入門(Part1)"
emoji: "🦭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Julia", "azarashi"]
published: false
---

# 本日は

[計算物理 春の学校 ２０２３](https://hohno0223.github.io/comp_phys_spring_school2023/) に参加された皆さんお疲れ様でした. ここでは [GomalizingFlow.jl](https://github.com/AtelierArith/GomalizingFlow.jl) に対するチュートリアルを書きます.

https://github.com/AtelierArith/GomalizingFlow.jl

# GomalizingFlow.jl について

GomalizingFlow.jl は格子上の場の理論, 特にスカラー場 $\phi^4$ 理論, における flow-based サンプリングを用いた配位生成アルゴリズムを提供するパッケージです．

https://twitter.com/MLPhysJP/status/1560741857222467584?s=20

プログラミング言語 Julia で書かれたものです. 実装のベースになったのは下記のツイートで紹介されている論文です:

https://twitter.com/MLPhysJP/status/1554395703429906433?s=20

- [Flow-based generative models for Markov chain Monte Carlo in lattice field theory](https://arxiv.org/abs/1904.12072)
- [Introduction to Normalizing Flows for Lattice Field Theory](https://arxiv.org/abs/2101.08176) も参考にしました．

# 格子上の場の理論

場の理論， (または場の量子論)は素粒子物理学の世界を記述する際に用いられる理論です.
その中で格子上の場の理論と言った場合, 連続的な時空間を離散化し時空間を有限の格子点の集まりとみなします． 場はその格子を用いて定義されたもとで数値計算や理論を展開することになります．

この立場では下記で述べるような数学的な定式化が可能になり, コードとして落とし込めることができます.

$d$ を時空の次元とします. $L$ を正の整数とします. 各方向のサイズが $L$ である格子点の集合を次で定めます:

$$
\mathcal{L} \coloneqq \{n = (n_1, \dots, n_d) \mid n_\mu \in \{1,2,\dots, L\}, \mu \in \{1,\dots, d\} \} \subset \mathbb{R}^d
$$

この $\mathcal{L}$ が $d$ 次元の離散化された時空に対応します. ここではスカラー場 $\phi$ の場合のみを考えます.
スカラー場 $\phi$ は形式的には $n \in \mathcal{L}$ 上の関数 $\phi=\phi(n)$ですが, 実際にコードを読むときは格子 $\mathcal{L}$ の位置 $n$ に実数値 $\phi(n)$ が配置されているオブジェクトとみなした方が良いかもしれません.
プログラム上だと $d=2$ だと行列, $d=3, 4$ では多次元配列として解釈できます. このオブジェクトを配位(configuration)と呼び, これも単に $\phi$ と書くことにします. スカラー場 $\phi^4$ 理論の設定では配位 $\phi$ は次の分布に従って与えられます:

$$\phi \sim P[\phi] = \exp(-S[\phi])/Z$$

ここで $S$ はユークリッド作用と呼ばれるものです. ここでは配位 $\phi$ を与えたら実数値を与えるものだと思いましょう. 後で具体的な $S$ は後で与えます. $Z$ は分配関数とよばれ積分の規格化因子として定ります:

$$
Z = \int \mathcal{D}\phi \exp(-S[\phi]).
$$

ここで $\mathcal{D}\phi$ は

$$
\mathcal{D}\phi = \prod_{n\in\mathcal{L}} d\phi(n)
$$

のことを表します. 例えば 3 変数の関数 $f=f(x_1, x_2, x_3)$ を積分するときに

$$\int dx_1dx_2dx_3 f(x_1, x_2, x_3)$$

のような記号を使うと思いますが, その時の $dx_1dx_2dx_3$ を $\prod_{n} dx_n$ のように書いていると思ってください. $d\phi(n)$ は $n \in \mathcal{L}$ で添字づけられた変数だと思えば良いです.

放置していたユークリッド作用 $S$ の説明に戻ります. 配位 $\phi$ に対してスカラー場 $\phi^4$ 理論に対応する作用 $S[\phi]$ を次で定義します:

$$
S[\phi]= - \sum_{n\in\mathcal{L}}\phi(n)\partial^2\phi(n) + \sum_{n\in \mathcal{L}} V[\phi](n)
$$

ここで, $V[\phi]$ はポテンシャル項と呼ばれ $n\in\mathcal{L}$ に対して

$$V[\phi](n) = m^2\phi(n) + \lambda \phi^4(n)$$

で与えられるものだとします. $m^2$ や $\lambda$ は定数です. 文献によっては $1/2$ や $1/4!$ をつけることが多いと思いますがここではつけていません.

$\partial^2\phi$ は離散ラプラシアンです. 具体的には下記のようになります:

$$
\partial^2\phi(n) = \sum_{\mu=1}^d(\phi(n + \hat{\mu}) + \phi(n - \hat{\mu}) - 2\phi(n)).
$$

さらに $n + \hat{\mu}$ の記号の説明も必要です. $n$ は格子 $\mathcal{L}$ の点であったことに注意しましょう. $n+\hat{\mu}$ は $d$ 次元空間の $\mu$ 軸の方向に対し $n$ を出発点とし一歩進んだ点を表します. $n - \hat{\mu}$ は $n$ を出発点とし $\mu$ 軸のマイナス方向に一歩進んだ点だとします. なお, 格子 $\mathcal{L}$ には境界条件は周期境界条件を採用することにします.

これで格子上のスカラー $\phi^4$ 理論を導入することができました. 定数 $d, L, m^2, \lambda$ を与えればそこから機械的に上記の定めた定義式を求め，配位の分布の定義を数学的に(形式的にかつ厳密に)与えられることがわかるでしょう.

時空間を格子にせず連続したまま取り扱うと $Z$ の定義式は無限次元の多重積分になります. つまり $\mathcal{D}\phi$ は

$$
\mathcal{D}\phi = \prod_{x \in \mathbb{R}^d} \phi(x)
$$

という計算を行う必要が出てきます. このような問題点を回避するために格子の場の理論が有効です. また格子上で考えることで上記で述べたように理論を形式的に定義することができるのも魅力的です. この時点で場の理論そもそもわからずチンプンカンプンになってるかもしれませんが， とりあえず問題設定を形式的に理解することができれば OK です.

# 格子上の場の理論における計算

ところで, 場の理論は量子力学と特殊相対性理論を土台にした理論です. 特に量子論の要請から物理量 $O$ は確率的に振る舞います.

$$\langle O \rangle = \int \mathcal{D}\phi P[\phi]O[\phi]$$

によって期待値を得ます. どうやらこの計算ができれば人類は幸せになれるようです.

$P[\phi]$ がなんだったかを思い出すには $Z$ と $S$ が分かればよかったです. $S$ に関しては

$$
S[\phi]= - \sum_{n\in\mathcal{L}}\phi(n)\partial^2\phi(n) + \sum_{n\in \mathcal{L}} V[\phi](n)
$$

で与えられるものでした. 配位 $\phi$ は格子上の点 $n\in \mathcal{L}$ に対して実数値が乗っかっているオブジェクトとおもえばOKでした. あとは右辺の式一つ一つを思い出せば機械的に計算できるようになってます. ここでの機械的というのは適切な定式化のもとでプログラムに落とし込めるという意味です. ひとまず理論物理がチンプンカンプンでも色々割り切って積分を数値計算を行えば人類に貢献できそうという一つの拠り所ができたのではないでしょうか.

# 高次元の積分の壁

上記の方針は間違ってないですが， 現実世界で計算機を走らせると大変なことになります. $d=4, L=10$ の場合, 格子点は $L^d = 10000$ 点になります. 我々が考えている積分の定義上, その数を変数とした多次元の多重積分を実行していることになります.（とても膨大ですが非加算無限の濃度の変数に関する多重積分を考えるよりはマシですね）.

この規模の高次元積分を台形公式などを使って真面目にすると現実時間では終わらないことが知られています([富谷さんの格子QCDの講義ノートを参照](https://hohno0223.github.io/comp_phys_spring_school2023/)).

困った. この手の計算では乱数を用いたモンテカルロ法をよく使います.

$$\langle O \rangle = \int \mathcal{D}\phi P[\phi]O[\phi]$$

を計算するために分布 $P$ に従う配位をサンプルします. 例えば $\phi_1, \phi_2, \dots, \phi_N$ としましょう.
$\langle O \rangle$ の計算を下記によって実現させることにします:

$$
\frac{1}{N}\sum_{i=1}^N O[\phi_i]
$$

この平均値のの期待値は $\langle O \rangle$ と一致し 誤差は $1/\sqrt{N}$ のオーダで減少することがわかります. 

ひとまず, 配位をいっぱい生成する手段が確立できれば積分対象の次元の高さに依らず
生成された配位から定まる物理量の平均を計算することに集中すれば良いことになります. 

# マルコフ連鎖モンテカルロ法による配位の生成

格子上の場の理論のテキストを読むと配位の生成のやり方としてマルコフ連鎖モンテカルロ法 (MCMC) の一つである ハミルトニアンモンテカルロ法(HMC) がよく使われます. [LatticeQCD.jl](https://github.com/akio-tomiya/LatticeQCD.jl) の README に書いてあるように LatticeQCD.jl は配位生成アリゴリズムとして HMC を採用した機能を提供しています. 自己学習モンテカルロ法 (SLMC) を使った手法も提供しているようです. SLMC については LatticeQCD.jl の開発者の一人である永井さんの解説 [自己学習モンテカルロ法：機械学習を用いたマルコフ連鎖モンテカルロ法の加速](https://www.jstage.jst.go.jp/article/mssj/21/1/21_15/_pdf/-char/ja) がとてもわかりやすいです. GomalizingFlow.jl も配位生成をする際に SLMC を使います.

簡単にですが, マルコフ連鎖モンテカルロ法を思い出します. これは点列

$$
x_0, x_1, x_2, \dots, x_k, x_{k+1}, \dots
$$

を逐次構成し所望の分布 $P=P(x)$ に収束させるアルゴリズムでした. $x_{k+1}$ は $x_{k}$ にのみによって決まります. ある $x$ から $x^\prime$ へ遷移する確率を $T(x \rightarrow x^\prime)$ のようにして書くとしましょう. 遷移確率と呼ばれる $T$ は所望の分布 $P$ との詳細つりあい条件

$$
P(x) T(x \rightarrow x^\prime) = P(x^\prime) T(x^\prime \rightarrow x)
$$

を満たすように設計することで関数 $O=O(x)$ の期待値を 

$$
\langle O \rangle = \int dx\ O(x) P(x) = \lim_{K\to\infty}\frac{1}{K}\sum_{k=1}^K f(x_k)
$$

によって得ることができるものでした. メトロポリス・ヘイスティング法は遷移確率 $T$ を提案確率 $f(x \to x^\prime)$ と 採択確率(または受理確率) $A( x \to x^\prime)$ に分解します:

$$
T(x\to x^\prime) = f(x \to x^\prime)A( x \to x^\prime)
$$

$A( x \to x^\prime)$ は詳細つりあい条件を満たすように

$$
A( x \to x^\prime) = \min\left(1, \frac{P(x^\prime) f(x^\prime \to x)}{P(x)f(x \to x^\prime)}\right)
$$

のようにして与えられます. アルゴリズムの $k$ ステップ目において $x_k$ があるとした時に $f$ 君が $x_{k+1}$ を「作ってええか？」と $A$ さんに尋ねます. $A$ さんは $\min$ の中にある式を計算の計算次第で「ええよ！ $f$ 君がくれた $x_{k+1}$ を採択するよ！」, 「やだ! $x_{k+1}$ は $x_k$ にするもん！」という採択または棄却するやりとりが聞こえるようになります. [ゼロからできるMCMC](https://www.kspub.co.jp/book/detail/5201749.html) に触れられているようにメトロポリス法, ギブスサンプリング, HMC などは今述べたメトロポリス・ヘイスティングの特別な場合と理解することもできます. 

# 自己相関の壁

さて，採択する確率が低い(棄却される頻度が多い)と

$$
x_0, x_1, x_2, \dots, x_k, x_{k+1}, \dots x_N
$$

という点列の間に相関が出てきます. これは次の意味で人間にとって都合が悪くなります.（極端な例として）例えば `1,1,4,4,2,2,3,3` として得られたデータの平均を求めたいとします. 偶数版目のデータは 一つ手前のデータと相関があります. これは実質 `1,4,2,3` という独立に得た 4 点のデータから求めるのと同じ程度の精度しか得られません. 得られる乱数の質が悪いと所望の誤差内にて計算するのに必要なサンプル数を増やす必要があります. 

誤差はルートNのオーダーで制御できます. これは精度を１桁改善するには必要なサンプル数の桁を2つ増やす必要を意味します:

```julia
julia> 1/sqrt(100)
0.1

julia> 1/sqrt(10000)
0.01

julia> 1/sqrt(1000000)
0.001
```

したがって相関が少ないサンプル方法が望まれています. [ゼロからできるMCMC](https://www.kspub.co.jp/book/detail/5201749.html) でも例を交えて説明しています. イジング模型で相転移点に近いパラメータで計算した際に自己相関が長くなるケースを示しています. これを臨界減速と呼ぶようです.

場の理論でも [富谷さんの格子QCDの講義ノート, スライドの94p 付近を参照](https://hohno0223.github.io/comp_phys_spring_school2023/) にて述べられているように相転移点が連続極限に対応しています. (どうやらそうらしいです. [格子上の場の理論(青木)](https://www.maruzen-publishing.co.jp/item/?book_no=294560) の繰り込み群と連続極限にその議論が書いてあります). 計算機上だと格子サイズ $L$ を増やすことに相当します.
経路積分の正当化のために時空を離散化したわけなので一旦格子で考えたら格子間の長さを狭める連続極限を考えるのは自然な要請です. ところが HMC などのマルコフ連鎖モンテカルロ法の効率と離散化誤差にはトレードオフが知られています. Flow-based な方法ではこの課題を解決するための手法の一つです.

まだ試していませんが, U(1) のゲージ理論だと [Equivariant flow-based sampling for lattice gauge theory](https://arxiv.org/pdf/2003.06413.pdf) の FIG. 1 のようにサンプリングが改善しているようです.

Flow-based の文脈では今考えているメトロポリス・ヘイスティング法の文脈では $A$ さんが「ええよ！」と採択する確率が 1 に近づくようにすれば改善できます.「これでええか？」と尋ねる提案確率 $f(x\to x^\prime)$ として目標とする分布 $P$ を近似する $\tilde{P}=\tilde{P}(x^\prime)$　を設計するアプローチを取ります. 

いい感じの $\tilde{P}$ を設計できれば採択確率 $A$ の式は

$$
A(x \to x^\prime) = \min\left(1, \frac{P(x^\prime)\tilde{P}(x)}{P(x)\tilde{P}(x^\prime)} \right)
$$

となります. $\min$ の 2 番目の引数が なるべく 1 になる (理想を言えば $P = \tilde{P}$) ようになれば良いわけです. 

GomalizingFlow.jl ではスカラー場のための $\tilde{P}$ の設計として Normalizing Flow による手法を実装しています.
ここまでくると場の理論の文脈は薄れ機械学習の文脈になります. 後述するように $\tilde{P}$ がニューラルネットワークを用いた機械学習モデルとなっており, 学習によりスカラー場の配位生成を可能とします.

# ボックス・ミュラー変換から学ぶ Flow-based models の気持ち

Flow-based の flow の気持ちに寄り添うためにボックス・ミュラー変換を思い出します. 下記のような $(x, y) \to (z_1, z_2)$ を考えます:

$$
\begin{aligned}
z_1 &= \sqrt{-2\log(x)} \cos (2\pi y), && \\
z_2 &= \sqrt{-2\log(x)} \sin (2\pi y).
\end{aligned}
$$

ここで $x$, $y$ は各々開区間 $(0, 1)$ を動くものとします. 

$r = \sqrt{-2\log(x)}, \theta = 2\pi y$ とみなせば $(z_1, z_2)$ は $(r, \theta)$ を媒介変数によって曲座標表示がされていると見做せます.
今 $x$ と $y$ が独立に区間 $(0, 1)$ 上の一様分布から得られたとき $z \coloneqq (z_1, z_2)$ はどのように振る舞うかを考えましょう. 確率変数 $X$ と対応する確率密度関数が $p_X = p_X(x)$ があるとします. $Y = f(X)$ というように変換された確率変数 $Y$ の密度関数 $P_Y = P_Y(y)$ はディラックのデルタ関数 $\delta$ を用いて次のように与えることができます:

$$
p_Y(y) = \int \delta(y - f(x)) p_X(x)\, dx\ .
$$

直感的には $f$ の値域に属する $y$ が与えられたとき $y = f(x)$ を満たす ($f$ の定義に属する) $x$ をかき集めてその $x$ から定まる値 $p_X(x)$ を足す(積分) することになります. 

特に $f$ が可逆(逆写像 $x = f^{-1}(y)$ を定義できる)であり(ヤコビ行列の行列式 $\partial f^{-1}/\partial y$ が定義できるという意味で)微分可能だとします. 多重積分の置換積分の公式を思い出せば $x = f^{-1}(\tilde{y})$ という変換で形式的に

$$
\begin{aligned}
p_Y(y) &= \int \delta(y - f(x)) p_X(x)\, dx \\
       &= \int \delta(y - \tilde{y}) p_X(f^{-1}(\tilde{y})) |\partial f^{-1}(\tilde{y})/\partial \tilde{y}|\, d\tilde{y} \\
       &= p_X(f^{-1}(y)) |\partial f^{-1}(y)/\partial y|
\end{aligned}
$$

と変形できます. $|\bullet|$ は絶対値を表します. この密度関数の関係式は $X$ やその密度関数 $P_X$ が人間にとって(解析的に記述できるという意味で)容易だけれど $Y$ の(分布の形状が複雑で)取り扱いに難しい時に役に立ちます. 

Normalizing Flow はこのような可逆な変換 $f$ をいくつか繰り返すことで単純な形状の密度関数から出発し密度関数を得る(分布を得る)方法です. $f$ を flow と呼びます. $f$ には学習パラメータが付いており学習によって適切な flow を作ることになります.

そして GomalizingFlow.jl のパッケージ名の由来になってます. 読者は Normalizing Flow の方を覚えてください. このページの読者は全体を通じて十分なユーモアセンスを持っていると仮定します.

以下では変数を $Y = f(X)$ のように変換させるという操作を反対方向から捉えることにします. すなわち, $f$ に注目する代わりに $g \coloneqq f^{-1}$ に注目し $Y$ を $X = g(Y)$ という変換で簡単な $X$ に帰着させるという立場をとることにします. [GomalizingFlow.jl に対応するプレプリント](https://arxiv.org/abs/2208.08903) では $g$ を自明化写像 (trivializing map) と呼んでいます.

このようにすると変数変換 $f$ から定まる密度関数の変換式は $p_Y(y) = P_X(g(y)) |\partial g(y)/\partial y|$ と書き換えられます. ${\bullet}^{-1}$ を省くことで記述が楽になります. 

もう一度ボックス・ミュラー変換の話に戻って $z = (z_1, z_2)$ の分布を調べることにしましょう. 

$$
(x, y) \overset{f_1}{\longrightarrow} (r, \theta) \overset{f_2}{\longrightarrow} (z_1, z_2)
$$

と捉える代わりに自明化写像を主軸に考えます.

$$
(z_1, z_2) \overset{g_2}{\longrightarrow} (r, \theta) \overset{g_1}{\longrightarrow} (x, y)
$$

この時 $(z_1, z_2) \coloneqq f_2(r, \theta) \coloneqq (r\cos\theta, r\sin\theta)$ であり $(r, \theta) \coloneqq g_2(z_1, z_2) \coloneqq f_2^{-1}(z_1, z_2)$ を与えることができることに注意します. $(z_1, z_2)$ の密度関数 $p_Z=p_Z(z_1, z_2)$ は $(r, \theta)$ に関する密度関数 $P_\Phi = P_\Phi(r, \theta)$ を用いて

$$
\begin{aligned}
p_Z(z_1, z_2)
&= p_{\Phi}(g_2(z_1, z_2)) |\partial g_2 /\partial (z_1, z_2)|  \\
&= p_{\Phi}(g_2(z_1, z_2)) |\partial f_2 /\partial (r, \theta)| ^ {-1} \\
&= \frac{1}{r}p_{\Phi}(g_2(z_1, z_2))
\end{aligned}
$$

と与えることができます. ヤコビ行列の行列式は曲座標表示の計算でお馴染みのように

$$
\begin{aligned}
\partial f_2 /\partial (r, \theta)
= \frac{\partial(z_1, z_2)}{\partial (r, \theta)}
= \det \begin{bmatrix}
	\frac{\partial z_1}{\partial r} & \frac{\partial z_1}{\partial \theta} \\ \\
	\frac{\partial z_2}{\partial r} & \frac{\partial z_2}{\partial \theta}
\end{bmatrix}
= \det \begin{bmatrix}
	\cos\theta & \sin\theta \\
	-r\sin\theta & r\cos\theta
\end{bmatrix} 
= r
\end{aligned}
$$

で与えられます. 続いて $(r, \theta)$ と $(x, y)$ の関係を議論します. $(r, \theta) \coloneqq f_1(x, y) \coloneqq(\sqrt{-2\log x}, 2\pi y)$ のように定めていました. この逆写像である
自明化写像 $g_1$ は 

$$
(x, y) \coloneqq g_1(r, \theta) \coloneqq (\exp(-r^2/2), y/2\pi)
$$

で与えることができます. これによって $p_\Phi$ は $(x, y)$ における密度関数 $p_{(X, Y)}$ を用いて

$$
\begin{aligned}
p_{\Phi}(r, \theta) 
&= p_{(X, Y)}(g_1(r, \theta)) |\partial g_1/\partial(r, \theta)| \\
&= p_{(X, Y)}(\exp(-r^2/2), y/2\pi) \left |\det \begin{bmatrix} -r \exp(-r^2/2) & 0 \\ 0 & \frac{1}{2\pi} \end{bmatrix}\right| \\
&= p_X(\exp(-r^2/2)) p_Y(y/2\pi) \frac{1}{2\pi} r \exp(-r^2/2) \\
&= \frac{1}{2\pi} r \exp(-r^2/2)
\end{aligned}
$$

区間 $(0, 1)$ 上の一様分布 $U(0, 1)$ の密度関数の定義と $\exp(-r^2/2)$, $y/2\pi$ が $(0, 1)$ の範囲に収まることから $p_X$, $p_Y$ の値は 1 として得られ結果としてヤコビ行列の行列式の項が分布を特徴付けるようになります.
まとめると

$$
\begin{aligned}
p_Z(z_1, z_2) 
&= \frac{1}{r} p_\Phi(r, \theta) \\
&= \frac{1}{r}\frac{1}{2\pi} r \exp(-r^2/2) \\
&= \frac{1}{2\pi} \exp(-r^2/2) \\
&= \frac{1}{2\pi} \exp(-(z_1^2 + z_2^2)/2).
\end{aligned}
$$

これは二変数の標準正規分布の密度関数そのものです. このようにして一様分布から得たデータを使って正規分布に従うデータを生成することができます. いわゆるボックス・ミュラー法ですね.
ボックス・ミュラー変換は 2 つの flow からなる Normalizing Flow とみなせるでしょう.

# ヤコビ行列の行列式の計算の壁を解決

ところで, Normalizing Flow は単純な確率分布(例えば多次元の正規分布)からスタートしいくつかの flow を経て複雑な確率分布を得る手法でした. 格子上の場の理論の文脈であれば作用 $S$ から定まる $P(x) = \exp(-S(x))/Z$ またはそれを近似する $\tilde{P}(x)$ を構成するために使います. 実際, 下記で述べるように, 格子サイズ分の次元の正規分布を flow によって $\tilde{P}$ を作成することをしています.

ボックス・ミュラー変換を Normalizing Flow という立場からもう一度眺めてみましょう. この変換の良い性質として自明化写像の微分(ヤコビ行列)が簡単に計算できることがあります. つまり，密度関数の計算式

$$
p_Y(y) = p_X(g(y)) |\partial g(y)/\partial y|
$$

における $|\partial g(y)/\partial y|$ が大学１年生が習う微積分程度の知識と手計算で厳密な値を求められることです.
素直に Normalizing Flow による計算を実装しようとすると「適切な変換の実装」,「変換の微分」と得られた行列の「行列式の計算」を強いられることになります. 特に 2, 3 番目は計算コストが高く数値計算の観点から技術的な問題がありました.

[Flow-based generative models for Markov chain Monte Carlo in lattice field theory](https://arxiv.org/abs/1904.12072) では flow の作成に [Density estimation using Real NVP](https://arxiv.org/abs/1605.08803) で導入されている affine coupling layer を採用しています. 

いま配位 $\phi$ が 1 次元のベクトルとして並べた時に $D$ (便宜上偶数)次元になってるとします. $\phi$ を二つに分割します. 添え字が even なのか odd なのかで $\phi_e$, $\phi_o$ のように分けましょう. 次のようにして $i$ 番目の自明化写像 $\varphi = T_i^{o}(\phi)$ を定義します:

$$
\begin{aligned}
\varphi_e &= \exp(s_i(\phi_o)) \odot \phi_e + t_i(\phi_o) \\
\varphi_o &= \phi_o
\end{aligned}
$$

$\odot$ は要素ごと(element-wise)の積を表すとします. $\exp(\phi_o)$ も $\phi_o$ の要素ごとに $\exp$ を適用していると思ってください. $s_i$, $t_i$ は学習パラメータ $\theta_i$ を持つ機械学習モデルです. もう少し詳しくいうと画像でも使う畳み込みネットワークをいくつか並べた非線形変換になります.

このようにして得られる変換のヤコビ行列の行列式を計算すると行列の行と列の適当な置換によって対角成分に 1 またはベクトル $\exp(s_i(\phi_o))$ の成分が並んだ三角行列を構成することができます. $D=2, 4$ の場合で実際に手を動かしてみるとよいでしょう. $\det$ の計算は列，行の置換で符号を除いて同じになることと, 三角行列の行列式は対角成分の積を取れば良い性質を使うとヤコビ行列の行列式の絶対値は $\prod_j \exp(s_i(\phi_o))_j$ と等しくなります. ベクトル $\exp(s_i(\phi_o))$ の成分の積を計算するだけにフォーカスすれば良いわけです. ちなみにヤコビ行列の行列式の「絶対値」を求めれば良いので行と列の置換で生じる±の符号の心配は気にしなくて大丈夫です.

また $\varphi = T_i^o(\phi)$ は構成法から $\phi = (T_i^o)^{-1}(\varphi)$ を構成することができます.


$$
\begin{aligned}
\phi_e &= \exp(-s_i(\varphi_o)) \odot \varphi_e - t_i(\varphi_o) \\
\phi_o &= \varphi_o
\end{aligned}
$$

$\exp(a)$ の逆数が $\exp(-a)$ となることをうまく利用していますね.

$T_i^o$ およびその逆変換は配位の偶数番目値を更新し奇数番目の値は変えず変換されます. 奇数番目の値を変えて偶数番目の値は変えない変換 $T_i^e$ も同様に定義できます.

GomalizingFlow.jl では $T_{\bullet}^e$, $T_{\bullet}^o$ を交互に適用した自明化写像 $\mathcal{F}$を構成しその逆写像 $\mathcal{F}^{-1}$ によって単純な分布から物理的に意味のある分布 $P$ を近似する $\tilde{P}$ を構成しています. $\mathcal{F}^{-1}$ を非自明化写像(un-trivializing map)と呼ぶことにします.

なんだか難しそうなことを言っていて「よくわかんない」と感じるかもしれません. その場合はひとまず「affine coupling layer というレイヤーをいくつか積み上げたディープニューラルネットワークを定義したんだな・・・」と思っていただければOKです.

$\mathcal{F}^{-1}$ をプログラムで書くことになります.

# 学習法

非自明化写像つまり $\tilde{P}$ には学習パラメータが入っています. そのパラメータを勾配法で学習し

受容確率

$$
A(x \to x^\prime) = \min\left(1, \frac{P(x^\prime)\tilde{P}(x)}{P(x)\tilde{P}(x^\prime)}\right)
$$

がなるべく 1 に近くなることを目指します. 元々は自己相関長を改善するものですがそのためには受容確率を増やすことが必要時条件となります.

損失関数は shifted KullbackLeibler (KL) divergence を採用します:

$$
\begin{aligned}
D_{KL}(\tilde{P}||P) - \log Z
&=
\int \mathcal{D}\phi \tilde{P} (\log \tilde{P}(\phi) - \log P(\phi) - \log Z) \\
&=
\int \mathcal{D}\phi \tilde{P}(\log \tilde{P}(\phi) + S(\phi))
\end{aligned}
$$

shifted のココロは $-\log Z$ を付与することで $-\log P$ を計算する時に出てくる分配関数 $\log Z$ を打ち消すというものです.

上記の関数は $\int$ が入ってて難しそうな雰囲気がありますが, 次の近似計算で実現します:

$$
\frac{1}{M}\sum_{i=1}^M \left(\log \tilde{P}(\phi^{(i)}) + S(\phi^{(i)})  \right)
$$

$M$ はミニバッチサイズだと思ってください. $\tilde{P}(\phi^{(i)})$ は次のようにして計算できます:

$$
\tilde{P}(\phi^{(i)}) = r(\mathcal{F}(\phi^{(i)})) \left|\frac{\partial \mathcal{F}}{\partial \phi}\right|_{\phi=\phi^{(i)}}
$$

$r$ は格子点数と同じ次元の標準正規分布の密度関数です. $z^{(i)} = \mathcal{F}(\phi^{(i)})$ となる $z$ は
標準正規分布を生成するプログラムがあれば作れます. $\phi^{(i)}$ は非自明化写像 $\mathcal{F}^{-1}(z^{(i)})$ を計算すればOKです. これはプログラムでかけます. ヤコビ行列の行列式の絶対値の部分は affine coupling layer の議論でベクトルの要素を掛け合わせるプログラムを書けばOKです. つまり，上記の式は実装可能な量でありコードとして落とし込むことができます.

# これまでの流れをまとめる

詳細は置いておいてプログラムを組むと次のような流れになります:

- 格子上の場を定義します. 配位は多次元配列(時空 $d$ 次元配列)と思えます.
- 作用は配位を入力とする関数です.
- 非自明化写像を定義します. affine coupling layer によって作られるニューラルネットワークを実装します.
- 多次元配列の要素次元数の正規分布から出発し非自明化写像を計算しその点 $x$ における $\tilde{P}(x)$ を計算します.
- shifted KullbackLeibler (KL) divergence を計算, 勾配法で非自明化写像のパラメータを学習.
  - 俗にいうディープでポン！
- 学習後: $A(x \to x^\prime) = \min\left(1, \frac{P(x^\prime)\tilde{P}(x)}{P(x)\tilde{P}(x^\prime)}\right)$ を受容確率とするメトロポリスヘイスティング法によって $P(x) = \exp(-S(x))/Z$ に従うサンプルを作成する.

こんな感じの流れになってます.

# まとめ

GomalizingFlow.jl が理論上何をしているのかの解説を書きました.
