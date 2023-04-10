---
title: "GomalizingFlow 入門(Part1-付録)"
emoji: "🖊"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Julia", "azarashi"]
published: false
---

# 設定

ここでは 4 変数の affine coupling layer でのヤコビアンの計算を示す.

$$
\phi = \begin{bmatrix}
\phi_1 \\ \phi_2 \\ \phi_3 \\ \phi_4
\end{bmatrix} \in \mathbb{R}^4
$$

が変換によって

$$
\varphi = \begin{bmatrix}
\varphi_1 \\ \varphi_2 \\ \varphi_3 \\ \varphi_4
\end{bmatrix} \in \mathbb{R}^4
$$

として得られたとする:

$$
\begin{aligned}
\varphi_e &= \exp(s(\phi_o)) \odot \phi_e + t(\phi_o), \\
\varphi_o &= \phi_o
\end{aligned}
$$

ここで $s$, $t$ は $\phi_o$ を入力として受け付ける写像 $\mathbb{R}^2 \to \mathbb{R}^2$ である. $\phi_e$, $\varphi_e$, $\phi_o$, $\varphi_o$ などは添え字が偶数か奇数かで分類したものとする:

$$
\begin{aligned}
\phi_o = \begin{bmatrix} \phi_1 \\ \phi_3 \end{bmatrix} &,
\varphi_o = \begin{bmatrix} \varphi_1 \\ \varphi_3 \end{bmatrix}, \\
\phi_e = \begin{bmatrix} \phi_2 \\ \phi_4 \end{bmatrix} &,
\varphi_e = \begin{bmatrix} \varphi_2 \\ \varphi_4 \end{bmatrix}. \\
\end{aligned}
$$ 


この時ヤコビアン $\partial{\varphi_i}/\partial{\phi_j}$ の絶対値が計算したい. 変換を真面目に式で成分でかくと

$$
\begin{aligned}
	\varphi_1 &= \phi_1, \\
	\varphi_2 &= \exp(s(\phi_o)_1) \phi_2 + t(\phi_o)_1, \\
	\varphi_3 &= \phi_3, \\
	\varphi_4 &= \exp(s(\phi_o)_2) \phi_4 + t(\phi_o)_2 \\
\end{aligned}
$$

のようになる. $s(\phi_o)_i$ は $s(\phi_o)\in\mathbb{R}^2$ の $i$ 番目の成分である $t(\phi_o)_i$ についても同様. 偏微分の計算を真面目にすると次のようになる:

$$
\begin{aligned}
\frac{\partial \varphi_1}{\partial \phi_1} = 1 ,&\ 
\frac{\partial \varphi_1}{\partial \phi_2} = 0 ,&\ 
\frac{\partial \varphi_1}{\partial \phi_3} = 0 ,&\ 
\frac{\partial \varphi_1}{\partial \phi_4} = 0 ,&\ 
\\
\frac{\partial \varphi_2}{\partial \phi_1} = \star ,&\ 
\frac{\partial \varphi_2}{\partial \phi_2} = \exp(s(\phi_o)_1) ,&\ 
\frac{\partial \varphi_2}{\partial \phi_3} = \star ,&\ 
\frac{\partial \varphi_2}{\partial \phi_4} = 0 ,&\ 
\\
\frac{\partial \varphi_3}{\partial \phi_1} = 0 ,&\ 
\frac{\partial \varphi_3}{\partial \phi_2} = 0 ,&\ 
\frac{\partial \varphi_3}{\partial \phi_3} = 1 ,&\ 
\frac{\partial \varphi_3}{\partial \phi_4} = 0 ,&\ 
\\
\frac{\partial \varphi_4}{\partial \phi_1} = \star ,&\ 
\frac{\partial \varphi_4}{\partial \phi_2} = 0 ,&\ 
\frac{\partial \varphi_4}{\partial \phi_3} = \star ,&\ 
\frac{\partial \varphi_4}{\partial \phi_4} = \exp(s(\phi_o)_2) ,&\ 
\end{aligned}
$$

ここで $\star$ と書いた部分は後の計算で明示的に計算する必要のない項なので表記をサボっている.

$$
\frac{\partial \varphi}{\partial \phi} = 
\det 
\begin{bmatrix}
	1 & 0 & 0 & 0 \\
	\star & \exp(s(\phi_o)_1) & \star & 0 \\
	0 & 0 & 1 & 0 \\
	\star & 0 & \star & \exp(s(\phi_o)_2)
\end{bmatrix}
$$

さて，行列式は対象となる行列の行どうしの置換, 列どうしの置換では符号を除いて同じだった. 今回欲しい情報は $\det$ の絶対値だ!!! 入れ替えし放題である.
2, 3 列目を置換した後, 2, 3 行目を置換すると次のように変形できる:

$$
\begin{aligned}
\begin{bmatrix}
	1 & 0 & 0 & 0 \\
	\star & \exp(s(\phi_o)_1) & \star & 0 \\
	0 & 0 & 1 & 0 \\
	\star & 0 & \star & \exp(s(\phi_o)_2)
\end{bmatrix}
& \to
\begin{bmatrix}
	1 & 0 & 0 & 0 \\
	\star & \star & \exp(s(\phi_o)_1) & 0 \\
	0 & 1 & 0 & 0 \\
	\star & 0 & \star & \exp(s(\phi_o)_2)
\end{bmatrix}
\\
&	\to 
\begin{bmatrix}
	1 & 0 & 0 & 0 \\
	0 & 1 & 0 & 0 \\
	\star & \star & \exp(s(\phi_o)_1) & 0 \\
	\star & 0 & \star & \exp(s(\phi_o)_2)
\end{bmatrix}
\end{aligned}
$$

このようにして $\det$ を計算とする行列を下三角行列に変換できた. つまり対角成分のみを計算すれば良い. これによって $|\det \bullet|$ を計算すると 

$$
\prod_i\exp(s(\phi_o)_i)
$$

を計算すれば良い. つまり $\exp(s(\phi_o))$ をプログラムで記述してその成分の積を計算すれば良い. NumPy ユーザーであれば `np.prod`, `np.exp` などの部品を使えば容易に記述できる. もちろん Julia でも可能である.

