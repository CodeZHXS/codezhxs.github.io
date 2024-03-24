## [A - 幸运数](https://www.lanqiao.cn/problems/3491/learning/?page=1&first_category_id=1&name=幸运数)

???+ Note "解题思路"

$10^8$ 个数，暴力枚举即可。

??? Success "参考代码"

    === "C++"
    
        ```c++
        #include <iostream>
        #include <cstdio>
        #include <algorithm>
        #include <cstring>
    
        using namespace std;
    
        bool odd(int x)
        {
            string s = to_string(x);
            return s.size() & 1;
        }
    
        bool check(int x)
        {
            string s = to_string(x);
            int a = 0, b = 0;
            for(int i = 0, j = s.size() - 1; i < j; i++, j--)
                a += s[i] - '0', b += s[j] - '0';
            return a == b;
        }
    
        int main()
        {
            // int ans = 0;
    
            // for(int x = 1; x <= 100000000; x++)
            // {
            //     if(odd(x))
            //         x *= 10;
            //     ans += check(x);
            // }
            // cout << ans << endl;
            cout << 4430091 << endl;
            return 0;
        }
        ```

---

## [B - 有奖问答](https://www.lanqiao.cn/problems/3497/learning/?page=1&first_category_id=1&name=有奖问答)

???+ Note "解题思路"

定义 $dp[i][j]$：前 $i$ 次问题答完后，得到的分数为 $10j$ 的情况数。

转移只需要考虑第 $i$ 次是答对还是答错，答错的时候 $j = 0$，其他情况就是答对的。

因此有：
$$
dp[i][j] = \left\{
\begin{matrix}
\sum\limits_{k = 0}^{k=9} dp[i-1][k], & j = 0\\
dp[i-1][j-1], & j > 0
\end{matrix}
\right.
$$
答案就是 $\sum\limits_{i=0}^{i=30}dp[i][7]$

??? Success "参考代码"

    === "C++"
    
        ```c++
        #include <iostream>
        #include <cstdio>
        #include <algorithm>
        #include <cstring>
    
        using namespace std;
    
        const int maxn = 35;
        int dp[maxn][maxn];
    
        int main()
        {
            dp[0][0] = 1;
            for(int i = 1; i <= 30; i++)
            {
                for(int j = 0; j <= 10; j++)
                    dp[i][0] += dp[i-1][j];
                for(int j = 1; j < 10; j++)
                    dp[i][j] = dp[i-1][j-1];
            }
            int ans = 0;
            for(int i = 0; i <= 30; i++)
                ans += dp[i][7];
            cout << ans << endl;
            return 0;
        }
        ```

---

## [C - 平方差](https://www.lanqiao.cn/problems/3213/learning/?page=1&first_category_id=1&name=平方差)

???+ Note "解题思路"

高精度，用了别人的板子。

??? Success "参考代码"

    === "C++"
    
        ```c++
        #include <iostream>
        #include <cstdio>
        #include <algorithm>
        #include <cstring>
        #include <vector>
        #include <string>
    
        using namespace std;
    
        // filename:    bigint_tiny.h
        // author:      baobaobear
        // create date: 2021-02-10
        // This library is compatible with C++03
        // https://github.com/Baobaobear/MiniBigInteger
        #pragma once
    
        struct BigIntTiny {
            int sign;
            std::vector<int> v;
    
            BigIntTiny() : sign(1) {}
            BigIntTiny(const std::string &s) { *this = s; }
            BigIntTiny(int v) {
                char buf[21];
                sprintf(buf, "%d", v);
                *this = buf;
            }
            void zip(int unzip) {
                if (unzip == 0) {
                    for (int i = 0; i < (int)v.size(); i++)
                        v[i] = get_pos(i * 4) + get_pos(i * 4 + 1) * 10 + get_pos(i * 4 + 2) * 100 + get_pos(i * 4 + 3) * 1000;
                } else
                    for (int i = (v.resize(v.size() * 4), (int)v.size() - 1), a; i >= 0; i--)
                        a = (i % 4 >= 2) ? v[i / 4] / 100 : v[i / 4] % 100, v[i] = (i & 1) ? a / 10 : a % 10;
                setsign(1, 1);
            }
            int get_pos(unsigned pos) const { return pos >= v.size() ? 0 : v[pos]; }
            BigIntTiny &setsign(int newsign, int rev) {
                for (int i = (int)v.size() - 1; i > 0 && v[i] == 0; i--)
                    v.erase(v.begin() + i);
                sign = (v.size() == 0 || (v.size() == 1 && v[0] == 0)) ? 1 : (rev ? newsign * sign : newsign);
                return *this;
            }
            std::string to_str() const {
                BigIntTiny b = *this;
                std::string s;
                for (int i = (b.zip(1), 0); i < (int)b.v.size(); ++i)
                    s += char(*(b.v.rbegin() + i) + '0');
                return (sign < 0 ? "-" : "") + (s.empty() ? std::string("0") : s);
            }
            bool absless(const BigIntTiny &b) const {
                if (v.size() != b.v.size()) return v.size() < b.v.size();
                for (int i = (int)v.size() - 1; i >= 0; i--)
                    if (v[i] != b.v[i]) return v[i] < b.v[i];
                return false;
            }
            BigIntTiny operator-() const {
                BigIntTiny c = *this;
                c.sign = (v.size() > 1 || v[0]) ? -c.sign : 1;
                return c;
            }
            BigIntTiny &operator=(const std::string &s) {
                if (s[0] == '-')
                    *this = s.substr(1);
                else {
                    for (int i = (v.clear(), 0); i < (int)s.size(); ++i)
                        v.push_back(*(s.rbegin() + i) - '0');
                    zip(0);
                }
                return setsign(s[0] == '-' ? -1 : 1, sign = 1);
            }
            bool operator<(const BigIntTiny &b) const {
                return sign != b.sign ? sign < b.sign : (sign == 1 ? absless(b) : b.absless(*this));
            }
            bool operator==(const BigIntTiny &b) const { return v == b.v && sign == b.sign; }
            BigIntTiny &operator+=(const BigIntTiny &b) {
                if (sign != b.sign) return *this = (*this) - -b;
                v.resize(std::max(v.size(), b.v.size()) + 1);
                for (int i = 0, carry = 0; i < (int)b.v.size() || carry; i++) {
                    carry += v[i] + b.get_pos(i);
                    v[i] = carry % 10000, carry /= 10000;
                }
                return setsign(sign, 0);
            }
            BigIntTiny operator+(const BigIntTiny &b) const {
                BigIntTiny c = *this;
                return c += b;
            }
            void add_mul(const BigIntTiny &b, int mul) {
                v.resize(std::max(v.size(), b.v.size()) + 2);
                for (int i = 0, carry = 0; i < (int)b.v.size() || carry; i++) {
                    carry += v[i] + b.get_pos(i) * mul;
                    v[i] = carry % 10000, carry /= 10000;
                }
            }
            BigIntTiny operator-(const BigIntTiny &b) const {
                if (b.v.empty() || b.v.size() == 1 && b.v[0] == 0) return *this;
                if (sign != b.sign) return (*this) + -b;
                if (absless(b)) return -(b - *this);
                BigIntTiny c;
                for (int i = 0, borrow = 0; i < (int)v.size(); i++) {
                    borrow += v[i] - b.get_pos(i);
                    c.v.push_back(borrow);
                    c.v.back() -= 10000 * (borrow >>= 31);
                }
                return c.setsign(sign, 0);
            }
            BigIntTiny operator*(const BigIntTiny &b) const {
                if (b < *this) return b * *this;
                BigIntTiny c, d = b;
                for (int i = 0; i < (int)v.size(); i++, d.v.insert(d.v.begin(), 0))
                    c.add_mul(d, v[i]);
                return c.setsign(sign * b.sign, 0);
            }
            BigIntTiny operator/(const BigIntTiny &b) const {
                BigIntTiny c, d;
                BigIntTiny e=b;
                e.sign=1;
    
                d.v.resize(v.size());
                double db = 1.0 / (b.v.back() + (b.get_pos((unsigned)b.v.size() - 2) / 1e4) +
                                   (b.get_pos((unsigned)b.v.size() - 3) + 1) / 1e8);
                for (int i = (int)v.size() - 1; i >= 0; i--) {
                    c.v.insert(c.v.begin(), v[i]);
                    int m = (int)((c.get_pos((int)e.v.size()) * 10000 + c.get_pos((int)e.v.size() - 1)) * db);
                    c = c - e * m, c.setsign(c.sign, 0), d.v[i] += m;
                    while (!(c < e))
                        c = c - e, d.v[i] += 1;
                }
                return d.setsign(sign * b.sign, 0);
            }
            BigIntTiny operator%(const BigIntTiny &b) const { return *this - *this / b * b; }
            bool operator>(const BigIntTiny &b) const { return b < *this; }
            bool operator<=(const BigIntTiny &b) const { return !(b < *this); }
            bool operator>=(const BigIntTiny &b) const { return !(*this < b); }
            bool operator!=(const BigIntTiny &b) const { return !(*this == b); }
        };
    
        int main()
        {
            string a, b;
            cin >> a >> b;
            BigIntTiny A(a), B(b);
            BigIntTiny ans = A * A - B * B;
            cout << ans.to_str();
            return 0;
        }
        ```

---

## [D - 更小的数](https://www.lanqiao.cn/problems/3503/learning/?page=1&first_category_id=1&name=更小的数)

???+ Note "解题思路"

如果一个区间逆序的字典序更小，那么反转这个区间就能得到更小的数字。

暴力枚举所有的 $l$ 和 $r$，比较 $[l, r]$ 的正序和逆序的字典序（双指针即可），时间复杂度是 $O(n^3)$，所以需要优化最后一层。

-   如果 $s[l] < s[r]$，正序字典序更小。
-   如果 $s[l] > s[r]$​，逆序字典序更小。
-   如果 $s[l] = s[r]$，在双指针的过程中我们会同时向内移动双指针，于是就变成了判断 $[l+1, r-1]$ 的子问题。

理所当然的就有区间 $dp[l][r]$：如果 $[l, r]$ 的逆序更小，则为 $1$，否则为 $0$。

转移方程为：
$$
dp[l][r] = \left\{
\begin{matrix}
0, & l = r\\
0, & r-l=1 \land s[l] \le s[r]\\
1, & r-l=1 \land s[l] > s[r]\\
0, & r-l>1 \land s[l] < s[r]\\
1, & r-l>1 \land s[l] > s[r]\\
dp[i+1][j-1], & r-l>1 \land s[l] = s[r]\\
\end{matrix}
\right.
$$
按照区间 **dp** 的顺序枚举，时间复杂度 $O(n^2)$

??? Success "参考代码"

    === "C++"
    
        ```c++
        #include <iostream>
        #include <cstdio>
        #include <algorithm>
        #include <cstring>
    
        using namespace std;
    
        const int maxn = 5000 + 10;
        int n;
        char s[maxn];
        bool dp[maxn][maxn];
    
        int main()
        {
            cin >> s;
            n = strlen(s);
            int ans = 0;
            for(int i = 0; i + 1 < n; i++)
            {
                dp[i][i+1] = s[i] > s[i+1];
                ans += dp[i][i+1];
            }
            for(int k = 3; k <= n; k++)
                for(int l = 0, r = k-1; r < n; l++, r++)
                {
                    dp[l][r] = s[l] == s[r] ? dp[l+1][r-1] : s[l] > s[r];
                    ans += dp[l][r];
                }
            cout << ans << endl;
            return 0;
        }
        ```

---

## [E - 颜色平衡树](https://www.lanqiao.cn/problems/3504/learning/)

???+ Note "解题思路"

枚举所子树的时间复杂度是 $O(n^2)$，可以通过 $60\%$ 数据。

事实上这道题就是 [树上启发式合并 - OI Wiki (oi-wiki.org)](https://oi-wiki.org/graph/dsu-on-tree/#算法内容) 的模板题，几乎一摸一样，只需要统计每个颜色出现的次数，以及相同频率出现的次数即可，询问答案相当于相同的频率是否只有一个。时间复杂度就是 $O(nlogn)$。

??? Success "参考代码"

    === "C++"
    
        ```c++
        #include <iostream>
        #include <cstdio>
        #include <algorithm>
        #include <cstring>
    
        using namespace std;
    
        const int maxn = 2E5 + 10;
        const int maxm = 2E5 + 10;
        int head[maxn], edge[maxm], ne[maxm], idx = 1;
        void add(int u, int v)
        {
            edge[idx] = v;
            ne[idx] = head[u];
            head[u] = idx++;
        }
    
        struct Data {
            int cnt[maxn] = {}, freq[maxn] = {}, k = 0;
            void add(int col)
            {
                int f = cnt[col];
                if(f && --freq[f] == 0)
                    k--;
                f = ++cnt[col];
                if(freq[f] == 0)
                    k++;
                freq[f]++;
            }
            void del(int col)
            {
                int f = cnt[col];
                if(--freq[f] == 0)
                    k--;
                f = --cnt[col];
                if(!f)
                    return;
                if(freq[f] == 0)
                    k++;
                freq[f]++;
            }
            bool check()
            {return k == 1;}
        }d;
    
        int n, col[maxn], L[maxn], R[maxn], ord[maxn], sz[maxn], son[maxn], ti, ans;
    
        void dfs1(int u)
        {
            L[u] = ++ti;
            ord[ti] = u;
            sz[u] = 1;
            for(int i = head[u]; i; i = ne[i])
            {
                int v = edge[i];
                dfs1(v);
                sz[u] += sz[v];
                if(sz[v] > sz[son[u]])
                    son[u] = v;
            }
            R[u] = ti;
        }
    
        void dfs2(int u, bool keep)
        {
            for(int i = head[u]; i; i = ne[i])
            {
                int v = edge[i];
                if(v == son[u])
                    continue;
                dfs2(v, 0);
            }
            if(son[u])
                dfs2(son[u], 1);
            for(int i = head[u]; i; i = ne[i])
            {
                int v = edge[i];
                if(v == son[u])
                    continue;
                for(int j = L[v]; j <= R[v]; j++)
                    d.add(col[ord[j]]);
            }
            d.add(col[u]);
            ans += d.check();
            if(!keep)
            {
                for(int i = L[u]; i <= R[u]; i++)
                    d.del(col[ord[i]]);
            }
        }
    
    
        int main()
        {   
            cin >> n;
            for(int i = 1, f; i <= n; i++)
            {
                scanf("%d %d", &col[i], &f);
                add(f, i);
            }
            dfs1(1);
            dfs2(1, 0);
            cout << ans << endl;
            return 0;
        }
        ```

---
