## [A - 幸运数](https://www.lanqiao.cn/problems/3491/learning/?page=1&first_category_id=1&name=幸运数)

??? Note "解题思路"

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

??? Note "解题思路"

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

??? Note "解题思路"

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

## 

??? Note "解题思路"



??? Success "参考代码"

    === "C++"
    
        ```c++
    
        ```

---

## 

??? Note "解题思路"



??? Success "参考代码"

    === "C++"
    
        ```c++
    
        ```

---
