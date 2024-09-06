
## 刷算法题


简单题，主要是基础的数据结构，比如排序，链表，队列、栈及一些字符串的算法。中等题，会涉及BFS, DFS，动态规划，树等。

ACM 模式常用的输出输入函数

C 库函数

- 字符串：3，49，30
- 线性表：86，16，27，732
- 队列：641，406，899
- 栈：946，116，117，895
- 哈希表：61，729，25，554
- dfs：105，112，98，494，547，1254
- bfs：1091，1129，102，101，752


## 一些输入输出

输入示例
3 4
11 40
输出示例
7
51

```c
#include <stdio.h>
int main()
{
    int a, b;
    while (scanf("%d %d\n", &a, &b) != EOF)
    {
        int c = a + b;
        printf("%d\n", c);
    }
}
```





## 读取一个字符串

```c
#include <stdio.h>
#include <string.h>
 
int main()
{
    char str[1000];
    int a=0, i=0;
    while(scanf("%s",str) != EOF);
    a = strlen(str);
    printf("%d",a);
}
```

到空格、换行就停止了。

要读取一行包含空格的

```c
scanf("%[^/n]", str);
```




