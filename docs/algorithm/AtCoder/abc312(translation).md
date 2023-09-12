## [A - Chord](https://atcoder.jp/contests/abc312/tasks/abc312_a)

> 判断字符串 $S$ 是否等于 `ACE`, `BDF`, `CEG`, `DFA`, `EGB`, `FAC`,  `GBD` 之一

---



## [B - TaK Code](https://atcoder.jp/contests/abc312/tasks/abc312_b)

> 有一个 $N \times M$ 的矩阵，矩阵中只有 `#` 或 `.` ，问：有多少个如下形状的子矩阵
>
> ```
> ###.?????
> ###.?????
> ###.?????
> ....?????
> ?????????
> ?????....
> ?????.###
> ?????.###
> ?????.###
> ```
>
> 其中 `?` 可以是  `#` 或 `.`

---



## [C - Invisible Hand](https://atcoder.jp/contests/abc312/tasks/abc312_c)

> 有 $N$ 个卖家和 $M$ 个买家，每个卖家都有预期卖出价格 $A_i$ ，当价格大于或等于 $A_i$ 时卖家才愿意卖出。买个买家有预期买入价格 $B_i$ ，当价格小于或等于 $B_i$ 时买家才愿意买入。找出最小的数 $X$ ，使得愿意卖出的人数大于等于愿意买入的人数。

---



## [D - Count Bracket Sequences](https://atcoder.jp/contests/abc312/tasks/abc312_d)

> 给你一个长度为 $N$ 的字符串 $S$ ，$S$ 由 `(`， `)` 和 `?` 组成，你需要将 `?` 替换成 `(` 或 `)`，问：有多少种方法能够将 $S$ 替换成括号匹配的字符串。
>
> 数据范围：$N ≤ 3000$

---



## [E - Tangency of Cuboids](https://atcoder.jp/contests/abc312/tasks/abc312_e)

> 在三维空间中，有 $N$ 个 长方体，如果两个长方体有一面相邻，则称这两个长方体相邻。输出每个长方体与其他长方体相邻的数量。
>
> 数据范围：
>
> - $1 ≤ N ≤ 10^5$
>
> - $0 ≤ X, Y, Z ≤ 100$

---



## [F - Cans and Openers](https://atcoder.jp/contests/abc312/tasks/abc312_f)

> 有 $N$ 个物品，每个物品都有一个 $X_i$ 属性，物品有三种类型，
>
> - 易拉罐：获得之后可以直接打开，打开之后获得 $X_i$ 的分数。
> - 密封罐：需要使用开罐器才能打开，打开之后获得 $X_i$ 的分数。
> - 开罐器：用于打开密封罐，可以使用 $X_i$ 次。
>
> 问：从中选择 $M$ 个物品，最多能获得多少分数？
>
> 数据范围：$N ≤ 2 \times 10^5$

---



## [G - Avoid Straight Line](https://atcoder.jp/contests/abc312/tasks/abc312_g)

> 有一棵 $N$ 个节点的树，求出满足下列条件的三元组 $(i, j, k)$ 的数量：
>
> - $1 ≤ i < j < k ≤ N$
> - 树中不存在包含 $i$ ，$j$ ，$k$ 的简单路径。
>
> 数据范围：$N ≤ 2 \times 10^5$