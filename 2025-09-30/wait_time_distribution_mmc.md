---
title: M/M/c における待ち時間分布の要点（静的・重負荷含む）
math: true
---

# 前提と記法

- モデル: M/M/c（到着 Poisson(λ)、サービス時間 Exp(μ)、サーバ数 c、FCFS）。
- パラメータ: 到着率 `λ`、1サーバあたりのサービス率 `μ`、利用率 `ρ = λ/(c μ) < 1`（安定条件）。
- 以降、`c, λ, μ` は時間一定（静的）を仮定。

# Erlang C と定常確率

- 定常分布の正規化定数:
$$
P_0^{-1} = \sum_{n=0}^{c-1} \frac{1}{n!}\left(\frac{\lambda}{\mu}\right)^n
\; + \; \frac{1}{c!}\left(\frac{\lambda}{\mu}\right)^c \cdot \frac{c\mu}{c\mu - \lambda} \,.
$$

- 待つ確率（Erlang C）:
$$
\boxed{\; C 
= P(\text{wait}) 
= \frac{\dfrac{1}{c!}\left(\dfrac{\lambda}{\mu}\right)^c \dfrac{c\mu}{c\mu - \lambda}}{\sum_{n=0}^{c-1} \dfrac{1}{n!}\left(\dfrac{\lambda}{\mu}\right)^n 
\; + \; \dfrac{1}{c!}\left(\dfrac{\lambda}{\mu}\right)^c \dfrac{c\mu}{c\mu - \lambda}}\; } 
\quad (= \frac{(\lambda/\mu)^c}{c!(1-\rho)} P_0) 
$$

# 待ち時間分布（厳密）

- 点質量: `P(W = 0) = 1 - C`（待たない確率）
- 条件付き分布: `W | (\text{wait}) ~ Exp(c\mu - \lambda)`
- よって無条件の分布は「0 に質点 + 指数分布」の混合:
$$
\boxed{\; F_W(t) = P(W \le t) 
= 1 - C\, e^{-(c\mu - \lambda) t}\quad (t \ge 0)\;}
$$
（t=0 に `1-C` のジャンプがある）

- 平均待ち時間（Little＋Erlang C）:
$$
\boxed{\; \mathbb{E}[W] = W_q = \frac{C}{c\mu - \lambda} \;}
$$

# 分位点（p-quantile）

- 一般の p 分位 `w_p` は次で与えられる:
$$
w_p = \begin{cases}
0, & C \le 1-p \\
\displaystyle -\frac{1}{c\mu - \lambda}\, \ln\!\left(\frac{1-p}{C}\right), & C > 1-p
\end{cases}
$$

- 特に 95% 分位（p=0.95）:
$$
\boxed{\; W_{95} = \begin{cases}
0, & C \le 0.05 \\
\displaystyle -\frac{1}{c\mu - \lambda}\, \ln\!\left(\frac{0.05}{C}\right), & C > 0.05
\end{cases}\;}
$$

# 重負荷近似（ρ → 1−）

- ρ が 1 に近い（≒リクエストが非常に多い）と `C \to 1` に近づく。
- このとき
$$
F_W(t) = 1 - C e^{-(c\mu - \lambda) t} \;\approx\; 1 - e^{-(c\mu - \lambda) t},
\quad W_q = \frac{C}{c\mu - \lambda} \;\approx\; \frac{1}{c\mu - \lambda}.
$$
- つまり「待ち時間は平均 `W_q` の指数分布」という単純形は、`C \simeq 1`（=ほぼ必ず待つ）場合の近似としては妥当。ただし一般には `t=0` の質点 `1-C` を無視できない。

# 備考（応答時間 R = W + S）

- サービス時間 `S ~ Exp(\mu)`（サーバ選択に依らず）。
- 無条件分布は混合: `R = S`（確率 `1-C`）と `R = W'+S`（確率 `C`）の混合。
- 後者の和 `W'+S` はレート `a=c\mu-\lambda` と `b=\mu` の独立指数の和（ハイポ指数）で、
$$
P(W'+S \le t) = 1 - \frac{b e^{-a t} - a e^{-b t}}{b - a}\;\; (a \ne b).
$$
- 実務上の近似: `C \le 0.05` なら `R_{95} \approx \ln(20)/\mu`、それ以外は近似的に `R_{95} \approx W_{95} + \ln(20)/\mu` を使うと保守的に評価できる。

# 要点のまとめ

- 静的でも待ち時間は「0への質点＋指数」の混合分布。単一の指数分布ではない。
- 「P(W \le t) = 1 - e^{-t/W_q}」は `C=1` の極限でのみ一致。重負荷近似としては使えるが、一般には上の厳密式を用いる。

