<div class="sidebar-toggle">
  <div class="sidebar-toggle-line-wrap">
    <span class="sidebar-toggle-line sidebar-toggle-line-first"></span>
    <span class="sidebar-toggle-line sidebar-toggle-line-middle"></span>
    <span class="sidebar-toggle-line sidebar-toggle-line-last"></span>
  </div>
</div>
这里是[TOJ](http://acm.tju.edu.cn/toj/)刷题记录，无特别顺序，有空再整理

# 4136.   Counting Ones(Simple Mode)
计算1的个数
``` c++
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

int t, n;
int ar[305];

int main()
{
    scanf("%d", &t);
    while(t --) {
        scanf("%d", &n);
        for(int i = 0; i < n; i ++)
            scanf("%d", &ar[i]);
        int sum = 0;
        for(int i = 0; i < n; i ++) {
            if(ar[i] == 1) sum ++;
        }
        printf("%d\n", sum);
    }
    return 0;
}
```

# 4141.   Friendly Integer
gcd的应用
``` c++
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

int n;
int ar[305];

int gcd(int a, int b ) {
    return b == 0 ? a : gcd( b, a%b );
}

int main()
{
    while(scanf("%d", &n) != EOF) {
        for(int i = 0; i < n; i ++)
            scanf("%d", &ar[i]);
        for(int i = 0; i < n; i ++) {
            int f = 0;
            for(int j = 0; j < n; j ++) {
                int x = gcd(ar[i], ar[j]);
                if(x > 1)
                    f ++;
            }
            if(f == n)
                printf("%d\n", ar[i]);
        }
    }
    return 0;
}
```

# 4139.   Muxiaokui and Image
注意反斜杠
``` c++
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

int t, n;
char str[105];

int main()
{
    scanf("%d", &t);
    while(t --) {
        scanf("%d", &n);
        int sum = 0;
        for(int i = 0; i < n; i ++) {
            scanf("%s", str);
            if(str[0] == '\\' && str[1] != '\\')
                sum ++;
        }
        printf("%d\n", sum);
    }
}
```

# 4094.   Spiral matrix
``` c++
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

int n, m;
int ar[205][205];

int main()
{
    while(scanf("%d %d", &n, &m) != EOF) {
        int flag = 0;
        int x = 0, y = 0;
        int ln = n-1, lm = m-1;
        int cnt = 1;
        while(ln >= 0 && lm >= 0) {
            if(ln == 0 || lm == 0) {
                if(ln == 0)
                    for(int i = y; i <= y + lm; i ++)
                        ar[x][i] = cnt ++;
                else
                    for(int i = x; i <= x + ln; i ++)
                        ar[i][y] = cnt ++;
                break;
            }
            if(flag == 0) {
                for(int i = y; i < y + lm; i ++)
                    ar[x][i] = cnt ++;
                y = y + lm;
                flag = 1;
                continue;
            }
            if(flag == 1) {
                for(int i = x; i < x + ln; i ++)
                    ar[i][y] = cnt ++;
                x = x + ln;
                flag = 2;
                continue;
            }
            if(flag == 2) {
                for(int i = y; i > y - lm; i --)
                    ar[x][i] = cnt ++;
                y = y - lm;
                flag = 3;
                continue;
            }
            if(flag == 3) {
                for(int i = x; i > x - ln; i --)
                    ar[i][y] = cnt ++;
                y = y + 1;
                x = x - ln + 1;
                lm = lm - 2;
                ln = ln - 2;
                flag = 0;
                continue;
            }
        }

        for(int i = 0; i < n; i ++) {
            for(int j = 0; j < m-1; j ++)
                printf("%d ", ar[i][j]);
            printf("%d\n", ar[i][m-1]);
        }
    }
    return 0;
}

```
