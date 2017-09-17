---
layout: post
title: 电子科技大学第八届ACM趣味程序设计竞赛系列题解
category: ACM
tags: [ ACM, WriteUp, algorithm ]
comments: true
---

只参加了第二场和第四场，成绩比较惨。别人分开发过了我就整合一下发一下自己的解法。

施工完毕，欢迎检阅。

鉴于文章太长了，题目内容就不发了，自己戳超链接看吧。

<!-- more -->

给个目录好了……

1. [第一场（热身赛）](#a-hrefhttpacmuestceducncontestshow130-targetblanka)
   1. [CDOJ 1 A+B Problem](#a-hrefhttpacmuestceducnproblemshow1-targetblanka---ab-problema)
   2. [CDOJ 977 中庸之道(一)](#a-hrefhttpacmuestceducnproblemshow977-targetblankb---a)
   3. [CDOJ 62 校门外的树](#a-hrefhttpacmuestceducnproblemshow62-targetblankc---a)
   4. [CDOJ 1014 The King and King boss](#a-hrefhttpacmuestceducnproblemshow1014-targetblankd---the-king-and-king-bossa)
   5. [CDOJ 762 铁路](#a-hrefhttpacmuestceducnproblemshow762-targetblanke---a)
2. [第二场（正式赛）](#a-hrefhttpacmuestceducncontestshow136-targetblanka)
   1. [CDOJ 1511 阴阳师？这游戏没有ssr！](#a-hrefhttpacmuestceducnproblemshow1511-targetblanka---ssra)
   2. [CDOJ 1512 可怜的非洲银](#a-hrefhttpacmuestceducnproblemshow1512-targetblankb---a)
   3. [CDOJ 1513 简单的数学题](#a-hrefhttpacmuestceducnproblemshow1513-targetblankc---a)
   4. [CDOJ 1523 我想上厕所](#a-hrefhttpacmuestceducnproblemshow1523-targetblankd---a)
   5. [CDOJ 1508 Megumin的爆裂魔法](#a-hrefhttpacmuestceducnproblemshow1508-targetblanke---megumina)
3. [第三场（正式赛）](#a-hrefhttpacmuestceducncontestshow139-targetblanka)
   1. [CDOJ 1510 渐变字符串](#a-hrefhttpacmuestceducnproblemshow1510-targetblanka---a)
   2. [CDOJ 1515 保护果实](#a-hrefhttpacmuestceducnproblemshow1515-targetblankb---a)
   3. [CDOJ 1514 Little_Pro的driver朋友们和他的魔法](#a-hrefhttpacmuestceducnproblemshow1514-targetblankc---littleprodrivera)
   4. [CDOJ 1509 扑克斗争](#a-hrefhttpacmuestceducnproblemshow1509-targetblankd---a)
   5. [CDOJ 1516 shallowdream and girl](#a-hrefhttpacmuestceducnproblemshow1516-targetblanke---shallowdream-and-girla)
4. [第四场（正式赛）](#a-hrefhttpacmuestceducncontestshow143-targetblanka)
   1. [CDOJ 1519 Picking&Dancing](#a-hrefhttpacmuestceducnproblemshow1519-targetblanka---pickingdancinga)
   2. [CDOJ 1520 string](#a-hrefhttpacmuestceducnproblemshow1520-targetblankb---stringa)
   3. [CDOJ 1518 How To Get Twenty-four?](#a-hrefhttpacmuestceducnproblemshow1518-targetblankc---how-to-get-twenty-foura)
   4. [CDOJ 1521 Passing the Ball](#a-hrefhttpacmuestceducnproblemshow1521-targetblankd---passing-the-balla)
   5. [CDOJ 1507 Homuras Game](#a-hrefhttpacmuestceducnproblemshow1507-targetblanke---homuras-gamea)

## <a href="http://acm.uestc.edu.cn/#/contest/show/130" target="_blank">第一场（热身赛）</a>

没有奖品就没去做，于是全是原题。

### <a href="http://acm.uestc.edu.cn/#/problem/show/1" target="_blank">A - A+B Problem</a>

题解？没有，给我滚.jpg

{% highlight cpp %}
#include <cstdio>
using namespace std;

int main() {
  int a, b;
  scanf("%d%d", &a, &b);
  printf("%d\n", a + b);
  return 0;
}
{% endhighlight %}

### <a href="http://acm.uestc.edu.cn/#/problem/show/977" target="_blank">B - 中庸之道\(一\)</a>

排序一下即可，注意分类讨论。据说题目最后不输出回车会导致PE，不过还好我有输出回车的习惯，USACO强行灌输的习惯

{% highlight cpp %}
#include <cstdio>
#include <algorithm>
using namespace std;

int T, n[3];

int main() {
  for (scanf("%d", &T); T; --T) {
    scanf("%d%d%d", &n[0], &n[1], &n[2]);
    sort(n, n + 3);
    printf("%d\n", n[1 + (n[0] == n[1] || n[1] == n[2])]);
  }
  return 0;
}
{% endhighlight %}

### <a href="http://acm.uestc.edu.cn/#/problem/show/62" target="_blank">C - 校门外的树</a>

{::nomarkdown}最简单的区间染色，只有一种颜色，数据小得可怜，官方标算是\(\Theta\left(n^2\right)\)的解法，不过有\(\Theta(n)\)的方法，先逆前缀和，再前缀和一次的方法。当然要是够闲的话直接线段树也不是不行。{:/nomarkdown}

{::nomarkdown}这里只给出我自己的\(\Theta(n)\)的方法。{:/nomarkdown}

{% highlight cpp %}
#include <cstdio>
using namespace std;

int L, M, s, t;
int road[10005], top, total;

int main() {
  scanf("%d%d", &L, &M);
  for (int i = 0; i < M; ++i) {
    scanf("%d%d", &s, &t);
    ++road[s]; --road[t + 1];
  }
  for (int i = 0; i <= L; ++i)
    if (!(top += road[i])) ++total;
  printf("%d\n", total);
}
{% endhighlight %}

### <a href="http://acm.uestc.edu.cn/#/problem/show/1014" target="_blank">D - The King and King boss</a>

{::nomarkdown}背包、DP不可用，\(\Theta\left(n^2\right)\)就会超时，所以其实，这题是\(\Theta(1)\)结论题。{:/nomarkdown}

{::nomarkdown}首先前缀和，前缀和数列中对\(n\)取模。在模中如果出现了\(0\)，那么直接从第一项到这一项构成一个子集其和可以被\(n\)整除。如果没有出现\(0\)，那么在\(n\)项中有\(1\)~\(n-1\)的可能，根据鸽巢原理肯定有两个重复项，那么这两个重复项中间所有元素构成的子集和就是\(n\)的倍数。{:/nomarkdown}

也就是说不论输入什么，输出都是`Yes`。

{% highlight cpp %}
#include <cstdio>
using namespace std;

int main() {
  puts("Yes");
  return 0;
}
{% endhighlight %}

### <a href="http://acm.uestc.edu.cn/#/problem/show/762" target="_blank">E - 铁路</a>

{::nomarkdown}事实上除了无法连通只有两种情况，道路过剩和道路过稀少分类即可。无法连通的很好考虑。道路过剩的情况，就是\(n(n-1)/2\)，道路过稀的情况，首先可以保证每座城市只有两条路的能连通（显而易见），大于两条路的则往上添路，反正必然可以连通，所以最多的路数就是\(kn/2\)。{:/nomarkdown}

{% highlight cpp %}
#include <cstdio>
#include <algorithm>
using namespace std;

int T, n, k;

int main() {
  for (scanf("%d", &T); T; --T) {
    scanf("%d%d", &n, &k);
	if (k == 0 || k == 1 && n > 2) puts("0");
    else printf("%d\n", min(k, n - 1) * n >> 1);
  }
  return 0;
}
{% endhighlight %}

## <a href="http://acm.uestc.edu.cn/#/contest/show/136" target="_blank">第二场（正式赛）</a>

参加了并死在D题上，估计没死也不想去做E题。妈呀计算几何太可怕了。

所以我还是不知道之前D题哪里被坑……

### <a href="http://acm.uestc.edu.cn/#/problem/show/1511" target="_blank">A - 阴阳师？这游戏没有ssr！</a>

![](http://acm.uestc.edu.cn/images/problem/1511/201611092144472767.jpg)

数学题，不解释了……最后推导出来的公式

{::nomarkdown}$$\sum_{i=1}^N\frac{200Pi-Pi^2}{10000}$${:/nomarkdown}

{% highlight cpp %}
#include <cstdio>
using namespace std;

int N, Pi;
double ans;

int main() {
  for (scanf("%d", &N); N; --N) {
    scanf("%d", &Pi);
    ans += (200.f - Pi) * Pi;
  }
  printf("%.3lf", ans / 10000.f);
}
{% endhighlight %}

### <a href="http://acm.uestc.edu.cn/#/problem/show/1512" target="_blank">B - 可怜的非洲银</a>

模拟题，直接模拟即可。不过这题题意不清，坑死了。

{% highlight cpp %}
#include <cstdio>
using namespace std;

int N, K, i;
int tot[10];
char c;

int main() {
  scanf("%d%d\n", &N, &K);
  for (i = 0; i < N && i < K; ++i) {
    if (++tot['F' - 'A'] == 50) {
      printf("%d F\n", i + 1);
      return 0;
    }
    while ((c = getchar()) != '\n') {
      if (++tot[c - 'A'] == 50) {
        printf("%d %c\n", i + 1, c);
        return 0;
      }
    }
  }
  if (i == K) puts("Feizhou Yin");
  else if (i == N) puts("AMNZ");
  return 0;
}
{% endhighlight %}

### <a href="http://acm.uestc.edu.cn/#/problem/show/1513" target="_blank">C - 简单的数学题</a>

我们知道只要交换两位和原数相差倍数就是9的倍数，然后要求输出最小的那个，直接排序一下即可。输入保证每一位都不是0，结果我原代码是判断过的。

{% highlight cpp %}
#include <algorithm>
#include <cstdio>
#include <cstring>
using namespace std;

char num[1005];
int length;

int main() {
  scanf("%s", num);
  sort(num, num + strlen(num));
  puts(num);
  return 0;
}
{% endhighlight %}

### <a href="http://acm.uestc.edu.cn/#/problem/show/1523" target="_blank">D - 我想上厕所</a>

不知道被什么坑了，不过这不重要。因为数据保证不会有三个人以上蹲错坑，所以分类讨论一下即可。

0人，0。2人，选择较短的那条路线一个一个交换过去。3人将环分成三段，最长的那条减掉再交换。

其实到最后也是公式题。

{% highlight cpp %}
#include <cstdio>
#include <algorithm>
using namespace std;

int n, loc;
int fls, p[3], dis[3];

int main() {
  scanf("%d", &n);
  for (int i = 0; i < n; ++i) {
    scanf("%d", &loc);
    if (--loc != i)
      p[fls++] = i;
  }
  switch (fls) {
  case 0: puts("0"); break;
  case 2:
    dis[0] = p[1] - p[0];
    dis[1] = n + p[0] - p[1];
    sort(dis, dis + 2);
    printf("%d\n", dis[0] * 2 - 1);
    break;
  case 3:
    dis[0] = p[1] - p[0];
    dis[1] = p[2] - p[1];
    dis[2] = n + p[0] - p[2];
    sort(dis, dis + 3);
    printf("%d\n", (n - dis[2]) * 2 - 2);
    break;
  }
  return 0;
}
{% endhighlight %}

### <a href="http://acm.uestc.edu.cn/#/problem/show/1508" target="_blank">E - Megumin的爆裂魔法</a>

有毒的计算几何，最小圆覆盖问题。然而最多只有三个点的凸包……这也算凸包？

一个点两个点不必说。三个点的时候，钝角三角形或者直角三角形或共线的情况下，求出最长边的中点为圆心，最长边长为直径，其他情况下，外心即可。

锐角三角形的时候，利用公式，三角形的外心为（见circumcenter函数）

{::nomarkdown}
$$\left(\frac{\begin{vmatrix} x_1^2+y_1^2 & y_1 & 1\\x_2^2+y_2^2 & y_2 & 1\\x_3^2+y_3^2 & y_3 & 1 \end{vmatrix}}{2\begin{vmatrix} x_1 & y_1 & 1\\x_2 & y_2 & 1\\x_3 & y_3 & 1 \end{vmatrix}},\frac{\begin{vmatrix} x_1 & x_1^2+y_1^2 & 1\\x_2 & x_2^2+y_2^2 & 1\\x_3 & x_3^2+y_3^2 & 1 \end{vmatrix}}{2\begin{vmatrix} x_1 & y_1 & 1\\x_2 & y_2 & 1\\x_3 & y_3 & 1 \end{vmatrix}}\right)$$
{:/nomarkdown}

半径为：

{::nomarkdown}
$$\frac{abc}{4\begin{vmatrix} x_1 & y_1 & 1\\x_2 & y_2 & 1\\x_3 & y_3 & 1 \end{vmatrix}}$$
{:/nomarkdown}

简直不想写……

{% highlight cpp %}
#include <cmath>
#include <cstdio>
#include <algorithm>
using namespace std;

typedef struct {
  double x, y;
} coordinate;

double sqr(double x) {
  return x * x;
}

double area(coordinate A, coordinate B, coordinate C) {
  return abs(A.x * (B.y - C.y) + B.x * (C.y - A.y) + C.x * (A.y - B.y)) / 2;
}

coordinate circumcenter(coordinate A, coordinate B, coordinate C) {
  double dist0 = A.x * A.x + A.y * A.y;
  double dist1 = B.x * B.x + B.y * B.y;
  double dist2 = C.x * C.x + C.y * C.y;
  double D = 2 * (A.x * (B.y - C.y) + B.x * (C.y - A.y) + C.x * (A.y - B.y));
  double Dx = dist0 * (B.y - C.y) + dist1 * (C.y - A.y) + dist2 * (A.y - B.y);
  double Dy = dist0 * (C.x - B.x) + dist1 * (A.x - C.x) + dist2 * (B.x - A.x);
  coordinate ret; ret.x = Dx / D; ret.y = Dy / D; return ret;
}

int n;
coordinate p[3], center;
double len[3], r;
int obtuse = -1, i, j, k;

int main() {
  scanf("%d", &n);
  for (int i = 0; i < n; ++i)
    scanf("%lf%lf", &p[i].x, &p[i].y);
  switch (n) {
  case 1: printf("0.0000 %.4lf %.4lf\n", p[0].x, p[0].y); break;
  case 2:
    r = hypot(p[0].x - p[1].x, p[0].y - p[1].y) / 2;
    center.x = (p[0].x + p[1].x) / 2; center.y = (p[0].y + p[1].y) / 2;
    printf("%.4lf %.4lf %.4lf\n", r, center.x, center.y);
    break;
  case 3:
    for (i = 0, j = 1; i < 3; ++i, ++j %= 3)
      len[3 ^ i ^ j] = hypot(p[i].x - p[j].x, p[i].y - p[j].y);
    for (i = 0, j = 1, k = 2; i < 3; ++i, ++j %= 3, ++k %= 3)
      if (hypot(len[i], len[j]) <= len[k]) { obtuse = k; break; }
    if (~obtuse) {
      center.x = (p[i].x + p[j].x) / 2; center.y = (p[i].y + p[j].y) / 2;
      printf("%.4lf %.4lf %.4lf\n", len[k] / 2, center.x, center.y);
    } else {
      coordinate center = circumcenter(p[0], p[1], p[2]);
      r = len[0] * len[1] * len[2] / area(p[0], p[1], p[2]);
      printf("%.4lf %.4lf %.4lf\n", r / 4, center.x, center.y);
    }
    break;
  }
  return 0;
}
{% endhighlight %}

## <a href="http://acm.uestc.edu.cn/#/contest/show/139" target="_blank">第三场（正式赛）</a>

因为去了厦门参加厦大组织的CTF，没有参加这场……下面都是现做的……

感觉这场也并不是能轻易完成的。

### <a href="http://acm.uestc.edu.cn/#/problem/show/1510" target="_blank">A - 渐变字符串</a>

{::nomarkdown}\(\Theta\left(n^2\right)\)暴力可过的数据，我的程序做了优化，我也不知道现在复杂度是多少……{:/nomarkdown}

让我想起了NOIP2015的斗地主，所以我至今还是不知道斗地主规则来着，不过这题规则简单，直接把能拼上的都拼上即可，所以我的程序就是每次找到一组最长连续的直接拿出来去掉，只要没找完接着找。

{% highlight cpp %}
#include <cctype>
#include <cstdio>
#include <algorithm>
using namespace std;

int n, tot[26], ans;
int s, t, m;
bool empty;

int main() {
  scanf("%d\n", &n);
  for (int i = 0; i < n; ++i)
    ++tot[getchar() - 'a'];
  while (s < 26) {
    ans += m;
    m = ~0u >> 1;
    for (; s < 26; ++s)
      if (tot[s] != 0) break;
    for (t = s; t < 26; ++t)
      if (!tot[t]) break;
      else m = min(m, tot[t]);
    while (t-- > s) tot[t] -= m;
  }
  printf("%d\n", ans);
  return 0;
}
{% endhighlight %}

### <a href="http://acm.uestc.edu.cn/#/problem/show/1515" target="_blank">B - 保护果实</a>

多边形的性质，只要最长边比其它边长度总和短就可以组成一个多边形。证明方法嘛……三角不等式。

{% highlight cpp %}
#include <cstdio>
using namespace std;

int n, a;
int m, s, i;

int main() {
  scanf("%d", &n);
  for (i = 1; i <= n; ++i) {
    scanf("%d", &a);
    if (a < m) s += a;
    else s += m, m = a;
    if (s > m) break;
  }
  if (i > n) puts("-1");
  else printf("%d\n", i);
  return 0;
}
{% endhighlight %}

### <a href="http://acm.uestc.edu.cn/#/problem/show/1514" target="_blank">C - Little_Pro的driver朋友们和他的魔法</a>

一开始看错了题目，以为是交换相邻两个元素，坑了我一个晚上。

显然的只有两种排列方法，分别是0开头和1开头，这个直接两个分别求出来取较小值即可，不必讨论。对于每一种排列，求出奇数位和偶数位分别有多少个不匹配，较小的可以和较大的那些交换，剩下的直接反相，所以变成这种排列的最小次数就是两者中的较大值。

然后去看了眼标程……这写得是什么玩意儿……原谅我嘲讽一下标程……

{% highlight cpp %}
#include <cstdio>
#include <algorithm>
using namespace std;

int n;
int m0[2], m1[2];

int main() {
  scanf("%d\n", &n);
  for (int i = 0; i < n; ++i) {
    bool drv = getchar() == '1';
    m0[i & 1] +=  drv ^ i & 1;
    m1[i & 1] += !drv ^ i & 1;
  }
  printf("%d\n", min(max(m0[0], m0[1]), max(m1[0], m1[1])));
  return 0;
}
{% endhighlight %}

### <a href="http://acm.uestc.edu.cn/#/problem/show/1509" target="_blank">D - 扑克斗争</a>

ACM的谓语是“玩”——对于题目的吐槽。

羡慕通信，羡慕狗头爸爸.jpg——对于题目的吐槽2。

貌似没什么好说的，不是很难地分类一下就可以了……两边都不傻，所以在z只有一张牌，或者两张牌、大的那张还比c最大那张要大的时候肯定能赢。

剩下的就是拼速度了，z已经出了一张牌，还剩一张，聪明人肯定留大的那张。这时c必须把所有比这张大的出完，如果剩下的卡中，只有一张比这个小的还好，如果出现两张就完蛋，于是观察倒数第二小的那张是不是比z最大的小即可。

{% highlight cpp %}
#include <cstdio>
#include <cstring>
using namespace std;

char z[3], c[14];
int lz, lc;

int w(char c) {
  switch (c) {
  case '3': case '4': case '5': case '6':
  case '7': case '8': case '9':
    return c - '2';
  case 'T': return 8;
  case 'J': return 9;
  case 'Q': return 10;
  case 'K': return 11;
  case 'A': return 12;
  case '2': return 13;
  }
}

int main() {
  scanf("%s%s", z, c); lz = strlen(z) - 1; lc = strlen(c) - 1;
  for (int i = 0; i <= lz; ++i) z[i] = w(z[i]);
  for (int i = 0; i <= lc; ++i) c[i] = w(c[i]);
  puts(lz && z[lz] < c[lc] && (!lc || c[1] >= z[lz]) ? "cfeitong" : "zhong_wang");
  return 0;
}
{% endhighlight %}

### <a href="http://acm.uestc.edu.cn/#/problem/show/1516" target="_blank">E - shallowdream and girl</a>

这！题！我！做！过！

还！写！过！题！解！

在哪儿来着我找找……

总之就是各种分类讨论。

`a1 >= 2`和`a1 >= 1 && a2 >= 1`的情况，可以填满所有的自不必说。

`a1 == 1 && a2 == 0`的情况，可以填满3k和3k+1另附一个1的情况。

`a1 == 0 && a2 >= 2 && a3 != 0`的时候，打表（误）可得，2至3&times;a3和3&times;a3+2至3&times;a3+2&times;a2的值。得到结果后反着就好证明了，其实就是拿下一个3替换上一个2或两个2，可以填完全部。（我真的是打表拿到的结果……）

剩下简单了。`a1 == 0 && a2 == 1`的时候，和前面的`a1 == 1 && a2 == 0`同理，不过这个是2和3组合……`a1 == 0 && a2 == 0`嘛不用说了。

找不到以前写的题解了。

这类题目多打表有帮助。

{% highlight cpp %}
#include <cstdio>
using namespace std;

long long a1, a2, a3;

long long posibility(long long a1, long long a2, long long a3) {
  if (a1 >= 2)      return 1 * a1 + 2 * a2 + 3 * a3 + 0;
  if (a1 == 1) {
    if (a2 != 0)    return 1 * a1 + 2 * a2 + 3 * a3 + 0;
    if (a2 == 0)    return 1 * a1 + 0 * a2 + 2 * a3 + 0;
  }
  if (a1 == 0) {
    if (a2 >= 2) {
      if (a3 != 0)  return 0 * a1 + 2 * a2 + 3 * a3 - 2;
      if (a3 == 0)  return 0 * a1 + 1 * a2 + 0 * a3 + 0;
    }
    if (a2 == 1)    return 0 * a1 + 1 * a2 + 2 * a3 + 0;
    if (a2 == 0)    return 0 * a1 + 0 * a2 + 1 * a3 + 0;
  }
}

int main() {
  scanf("%lld%lld%lld", &a1, &a2, &a3);
  printf("%lld\n", posibility(a1, a2, a3));
  return 0;
}
{% endhighlight %}

## <a href="http://acm.uestc.edu.cn/#/contest/show/143" target="_blank">第四场（正式赛）</a>

最后参加了的一场，对排名有点不服，不过也不能小瞧别人。正如我之前和某人所吹逼的：

> 我不一定比他们弱，没拿奖的也不一定比我们弱。

妈呀这场才是全结论题，可怕极了。

### <a href="http://acm.uestc.edu.cn/#/problem/show/1519" target="_blank">A - Picking&Dancing</a>

……难道不是判断奇偶性？我还思考了好久好久怀疑我题目看错了……

{% highlight cpp %}
#include <cstdio>
using namespace std;

int n;

int main() {
  scanf("%d", &n);
  puts(n & 1 ? "Xiaoyu_Chen" : "Yitong_Chen");
  return 0;
}
{% endhighlight %}

### <a href="http://acm.uestc.edu.cn/#/problem/show/1520" target="_blank">B - string</a>

其实就是从第一个字符开始的连续相同字符的字符串的长度。因为我们可以用第一个不同的把后面所有的都删除，再用前面的那个把那个不同的删光，剩下只有一开始那串字符全部相同的字符串。

{% highlight cpp %}
#include <cstdio>
using namespace std;

char f;
int total;

int main() {
  f = getchar();
  while (getchar() == f) ++total;
  printf("%d\n", total + 1);
  return 0;
}
{% endhighlight %}

### <a href="http://acm.uestc.edu.cn/#/problem/show/1518" target="_blank">C - How To Get Twenty-four?</a>

{::nomarkdown}一开始以为是四个\(N\)算24点来着……结果是\(N\)个7算24点……{:/nomarkdown}

样例给出了N=6的时候可以拼出24，于是所有大于等于6的偶数都可以拼出来，同理如果奇数的找到了一个那么也可以+7-7重新得到24。

{::nomarkdown}4个、5个7是拼不出来的。6个的时候(7&times;7&times;7-7)&divide;(7+7)=24，七个的时候(7&divide;7+7)&times;[(7+7+7)&divide;7]=24，所以其实只要N比5大即可。{:/nomarkdown}

所以又成了结论题……

{% highlight cpp %}
#include <cstdio>
using namespace std;

int N;

int main() {
  scanf("%d", &N);
  puts(N > 5 ? "YES" : "NO");
  return 0;
}
{% endhighlight %}

### <a href="http://acm.uestc.edu.cn/#/problem/show/1521" target="_blank">D - Passing the Ball</a>

因为数据小的可怜，所以据说暴搜也可过。我用了DP，结果标算给了个公式……

暴搜不放了，给个DP的思路，代码用公式好了。

`opt[i][j]`表示第`i`轮传到了第`j`个人手上的方法数。`sum[i]`表示第`i`轮的`opt`的和。于是`opt[i][j] = sum[i] - opt[i - 1][j]`。

接着优化，容易知道，或者说根据对称性，`opt[i]`除了`opt[i][0]`，也就是第一个人之外，其值全部相等，于是我们可以得到两个数 列，我设{::nomarkdown}\(a_i\)为第一个人在第\(i\)次拿到球的所有情况数，\(b_i\)为其他人的，则$$\begin{align*}a_i&=(n-1)b_{i-1}\\b_i&=(n-2)b_{i-1}+a_{i-1}\end{align*}$$解通项的话用特征方程，可以直接得到$$\begin{align*}b_i&=\frac{(n-1)^i-(-1)^i}n\\a_i&=\frac{n-1}n[(n-1)^{i-1}+(-1)^i]\end{align*}$${:/nomarkdown}

{% highlight cpp %}
#include <cmath>
#include <cstdio>
using namespace std;

long double N, M;

int main() {
  scanf("%Lf%Lf", &N, &M);
  printf("%.0Lf\n", (pow(N - 1, M - 1) + pow(-1, M)) * (N - 1) / N);
  return 0;
}
{% endhighlight %}

### <a href="http://acm.uestc.edu.cn/#/problem/show/1507" target="_blank">E - Homura's Game</a>

怎么……可以这么简单？怀疑了10min+的人生，然后提心吊胆提交了，直接AC（忘记去`while(1);`造成的TLE不算……尼玛被这个坑死了，应该在比赛前配好环境的。）

二维的大概都知道了，直接判断奇偶性就可以了，三维……同理。

证明的话，把立方染色成黑白两色，每一轮都会去掉一个黑一个白，不妨设一开始在白色，如果每次去掉一对可以把全部去掉（亦即偶数），也就是对方不能继续移动就胜利，不能去掉全部，也就是剩下一个白色的话（亦即奇数）我方输。

HTML注释里有一堆东西，愿意看的话……

<!-- 非要说的话……该图是一个二分图，偶数情况下（必胜态）为完美匹配，求一次最大匹配，我方每次沿着匹配边走，对方只能走 非匹配边，最后走到没路为止，也就是我方需要占据匹配边的起点。奇数情况下，去掉起点求一次最大匹配，则对方处于必胜态。

如果对手是随机走的话（对手足够傻_），需要在场面中每出现新的不可抵达的位置），统计可抵达区域的数量奇偶性确认自己是必胜态 还是必败态。也就是先手必须不停造圈儿，然而一般情况下，造圈会加速自己死亡，毕竟对手也没这么傻。
-->

{% highlight cpp %}
#include <cstdio>
using namespace std;

int a, b, c;

int main() {
  scanf("%d%d%d", &a, &b, &c);
  puts(a & b & c & 1 ? "NO" : "YES");
  return 0;
}
{% endhighlight %}

## 最后

结论题你好，结论题再见。

虽然第三场按照罚时拿到的是很靠后的名次，我被罚了整整七次，有三次是因为该死的`while (1);`吧，不过按照AK的时间来说还是第三，和第一差了大概4分钟，和后一名差了将近一小时……苦逼，没拿到一等有点不服。不过现在来说的我，也早就退化到了初学者的水 平了。

大约是我太久没做算法竞赛相关了，所以感觉手疏了很多，以及最近养成的，总是习惯性打工程代码，也许某方面来说是有利的，不过如果搞竞赛相关，那就算是一个弱势了。不论怎样，如果还打算在ACM界混的话，应该要重拾这个玩意儿了。

如果没有加工作室的话，也许我现在也还在刷着CF, TC呢……

以及最后收获一份奖状和两本书，怂……

![](/img/uestc-8th-fun-game.jpg)