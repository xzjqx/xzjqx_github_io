---
title: toj
date: 2016-04-16 20:59:05
---
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
