---
title: "KMP算法"
subtitle: ""
date: 2022-08-25T16:06:00+02:00
draft: false
author: "spike"
authorLink: ""
authorEmail: ""
description: ""
keywords: ""
license: ""
comment: true
weight: 0

tags:
- algorithms
categories:
- data structure and algorithms

hiddenFromHomePage: false
hiddenFromSearch: false

summary: ""
resources:
- name: featured-image
  src: featured-image.jpg
- name: featured-image-preview
  src: featured-image-preview.jpg

toc:
  enable: true
math:
  enable: false
lightgallery: false
seo:
  images: []

repost:
  enable: true
  url: ""

# See details front matter: https://fixit.lruihao.cn/theme-documentation-content/#front-matter
---



### KMP算法

我们知道，串类型有一个功能是查找子串，而查找的方法有多种，如暴力查找法、KMP算法，这次我们来介绍一下这些方法

<!--more-->

首先说明一下一些符号及名词的含义：

- S为主串
- T为待查找的子串，也称为模式串
- 子串在主串中的位置： 子串在主串中第一次出现时首字母在主串中的索引位置，即子串在主串中的位置。对于主串“to be or not to be ,This is a question”，子串“be”在子串中的位置为3
- 查找子串，即从主串中查找子串是否存在，如果存在返回子串在主串中的位置
- 字符串的前缀：对于字符串A和B，若有A=BS，B、S为任意非空字符串，则称B为A的前缀。如字符串“Pig”，其前缀包括{“P”, “Pi”}
- 字符串的后缀： 对于字符串A和B，若有A=SB，B、S为任意非空字符串，则称B为A的后缀，如字符串“Pig”，其后缀包括{“g”, “ig”}
- i、j 分别代表当前匹配到的主串位置和子串位置
- 此外，在本文中，数组的索引一律从0开始

#### 暴力查找

暴力查找的步骤为：

```C#
if (S[i] == P[j])
{
    i++;j++
}
else
{
    i = i - j + 1;
    j = 0;
}
```

其文字表述为：

1. 从主串的首字符开始，与子串的每个字符 逐一进行比较
2. 若遇到不匹配的字符，则将当前首字符的下一个字符作为首字符
3. 重复步骤1，步骤2，直到寻找到子串或者扫描完整个主串(其中，i、j表示主串和子串当前比较的位置)

举个栗子：如下图，我们要寻找“ABCE”在“ABCABCDHIJK”中的位置

![image-20220825160842266](/img/image-20220825160842266.png)

1. S[0] == P[0], 执行i++;j++ ，比较S[1]与P[1]

2. S[1] == P[1], 执行i++;j++ ，比较S[2]与P[2]

3. S[2] == P[2], 执行i++;j++ ，比较S[3]与P[3]

![image-20220825160858757](/img/image-20220825160858757.png)

4. S[3] != P[3], 执行 i = i - j + 1;j=0;  得到i=1, j=0，重新开始比较;

![image-20220825160909224](/img/image-20220825160909224.png)

5. S[1] != P[0],执行 i = i - j + 1;j=0;  得到i=2, j=0，重新开始比较;

6. S[2] != P[0],执行 i = i - j + 1;j=0;  得到i=3, j=0，重新开始比较;

7. S[3] == P[0],执行i++,j++;得到i=4，j=1，继续比较

8. S[4] == P[1],执行i++,j++;得到i=5，j=2，继续比较

9. S[5] == P[2],执行i++,j++;得到i=6，j=3，继续比较

   依次比较，得到结果…………..

我们不难看出上述的比较方式中存在着一些不必要的步骤：

当比较到第4步时主串和子串的前三位是匹配的（即S[0] == P[0], S[1]\==P[1],S[2]==P[2]），也就是说我们可以由模式串的前三位字符确定主串的前三位字符，很明显前三位字符分别为A、B、C。我们也很容易可以看出P[0] != P[1], P[0]!=P[2],所以有S[1] != P[0], S[2]!=P[0].

也就是说比较到第4步时，我们就知道了S[1] != P[0], S[2]!=P[0]，而按照暴力查找的步骤， 它会会依次比较S[1]和P[0], S[2]和P[0]，也就是说暴力查找进行了不必要的指针回溯（这里的指针即上文中的i与j），这很明显是多余的操作。



面对暴力查找这种机械又存在明显缺点的字符串查找算法，一般人可以忍，但D.E.Knuth，J.H.Morris和V.R.Pratt这三位大佬忍不了，他们提出了一种新算法，可以避免暴力查找算法中会出现的不必要的指针回溯问题，提高查找效率。这个算法就是我们接下来要介绍的KMP算法。



#### KMP算法

KMP算法的步骤是：

```C#
if (j == -1 || S[i] == P[j])
{
    i++;
    j++;
}
else
{
    j = next[j];
}
```

用文字来表述即：如果j==-1或者S[i]与S[j]匹配成功，那么执行i++,j++,匹配主串和子串的下一位。

如果S[i]与S[j] 匹配失败，i指针不变，j指针回溯到next[j]处。

这里的next[]是一个整型数组，也是KMP算法的核心

接下里我们就来说说next数组是什么及为什么上述的步骤可以避免暴力查找中不必要的指针回溯问题。

##### 寻找前缀后缀公共元素的最长长度

我们首先来寻找一下模式串的前缀后缀公共元素的最长长度

给定一个模式串“ABCDABD”，其子串的前缀、后缀、前后缀最大公共元素长度如下表：

![img](/img/20140725231726921.jpeg)

在说道next数组之前，我们先说一下另一个概念，部分匹配表。

##### 部分匹配表

同样是对于模式串“ABCDABD”，其部分匹配表如下：

![img](/img/20140721230517324.jpeg)

将部分匹配表看作一个数组a的话，其每一个位置上的值 就是模式串中  从字符串开头到该位置 的 子串其前后缀最大公共元素的长度。

例：a[2]位置的值即 模式串的子串“ABC”其前后缀最大公共长度值。“ABC”的前缀为{“A”,“AB”},“ABC”的后缀为{“C”,“BC”},其最大公共长度值为0，所以a[2] = 0;

##### next数组

那么我们的next数组究竟是什么呢？

如果我们把模式串“ABCDABD”的部分匹配表整体右移一位，并将首元素设为-1，就得到了我们的next数组。

![img](/img/20140728110939595.jpeg)

那么next数组每一位的值分别代表什么呢？

next[i] 的值 表示 P[0]-P[i-1]这个子串其  前缀后缀最大公共元素的长度。对于next[0]，由于它之前没有字符，将其值设为-1

再**举个栗子**：next[4]的值就是 模式串的子串“ABCD” 前后缀最大公共元素的长度 next[4] = 0



知道了next数组的具体含义，接下来我们就来看看KMP算法具体是如何执行的，以及为什么将j回溯到next[j]就可以避免暴力查找算法中不必要的指针回溯问题

我们来举个栗子：



给定主串“BBC ABCDAB ABCDABCDABDE”，模式串“ABCDABD”，寻找模式串在主串中的位置

模式串的next数组如下图：

![img](/img/20140728110939595.jpeg)

匹配过程如下：

![img](/img/20140723224710203.png)

1. i=0,j=0,S[0] != P[0],执行j=next[0],得到i=0,j=-1

2. i=0,j=-1,满足j==-1，执行i++,j++;得到i =1, j=0;
3. i=1,j=0, S[1]!=P[0],执行j=next[0],得到j=-1;
4. i=1,j=-1,满足j==-1，执行i++,j++;得到i =2, j=0;
5. i=2,j=0,S[2]!=P[0]，执行j=next[0],得到i=2，j=-1；
6. 重复以上步骤，直到下图位置

![img](/img/20140726213602848.png)

7. 此时匹配成功,i=4, j=0, S[4]==P[0],执行i++，j++;得到i=5,j=1;继续匹配至下图位置

![img](/img/20140721223809617.png)

8. 此时i=10，j=6，S[10]!=P[6],执行j=next[6],而next[6]=2,所以j指针回溯到2，得到i=10,j=2;从直观上来看，next[6]即模式串的子串“ABCDAB”的前后缀公共元素最大长度

   换个角度来看，next[6]即主串中“ ”前的子串“ABCDAB”的后缀与模式串子串“ABCDABD”的前缀 的公共元素最大长度，在这里为2，又由于数组、字符串的索引从0开始，那么2的位置恰好对应 公共元素的下一位，这样我们就避免了i、j指针不必要的回溯，利用已知的匹配信息，将j回溯到合理的位置

以上就是KMP算法的过程。

我们还遗漏了一个问题，如何求出next数组呢？代码如下

```C#
public int[] Get_Next(IString s)
{
    int i = 0;
    int j = -1;
    int[] next = new int[256];
    next[0] = -1;
    int patternLength = s.Length;
    while (i < patternLength)
    {
        if (j == -1 || s[i] == s[j])
            // s[i]表示后缀字符
            // s[j]表示前缀字符
        {
            ++i;
            ++j;
            next[i] = j; // next[i]之前模式串的前缀后缀最大匹配长度j
        }
        else
        {
            j = next[j];
            // 不匹配，则j回溯
            //模式字符串以kmp算法的思路 与自身匹配
        }
    }
    return next;
}
```

我们可以看到，获取next数组的代码和KMP算法查找的代码其实是很相似的。

我们可以理解为：将模式串P作为主串，将模式串P又作为模式串，以KMP算法的思路保持i不变对j进行回溯。很有意思的是，在KMP算法中我们利用next数组来回溯j，而在求next数组的过程，我们也利用next数组本身来回溯j。



我们来举个栗子，看看这段代码究竟是怎样工作的。

对于模式串P“ABAB”,其前缀集合为{A、AB、ABA},后缀集合为{B、AB、BAB}，交集为AB

1. i=0,j=-1,满足j=-1,执行i++，j++,next[i]=j;得到i=1,j=0,next[1]=0;
2. i=1,j=0，比较P[1]与P[0],P[1]!=P[0],执行j=next[j]，j=next[0]=-1,得到i=1,j=-1;
3. i=1,j=-1;满足j=-1,执行i++，j++,next[i]=j;得到i=2,j=0,next[2]=0;
4. i=2,j=0，比较P[2]与P[0],P[2]==P[0],执行i++,j++,next[i]=j,得到i=3,j=1，next[3]=1;

这样我们就得到了模式串“ABAB”的next数组，其元素集合为{-1,0,0,1}

我们也可以从前缀后缀的角度来求next数组





以上基本上算是介绍完了KMP算法，相对于暴力查找算法给出了一种更高效的字符串查找算法。

但是KMP算法也存在着缺漏。

我们再来举个栗子：



给定主串“abacababc”，模式串“abab”，模式串next数组的取值为-1、0、0、1

![image-20220825162019693](/img/1661437234045.jpg)

1. 当匹配到S[3]与P[3]时，S[3]!=P[3], i不变,j =next[3]=1,得到i=3，j=1
2. 对S[3]与P[1]进行匹配，S[3]!=P[1]. 但是我们注意到P[1]==P[3],而P[3]!=S[3],我们可以得出P[1]!=S[3].那么 j 回溯到next[3]=1这一步可以说是多余的。于是人们对KMP中next数组的求解进行了改进。

​      我们知道，当我们遇到主串和模式串的不匹配时，j会回溯到next[j]的位置，如果next[j]仍然不匹配，那么j会接着回溯，回溯到next[next[j]]处。

​	在这个过程中如果j与next[j]位置的值是相同的，那我们知道next[j]与主串也是不匹配的，我们 完全可以在回溯前判断一下j与next[j]处的值是否相同，避免不必要的回溯，如果next[j]与j处的值相同，那就将j回溯到next[next[j]]

```C#
public int[] Get_Next1(IString s)
        {
            int i = 0;
            int j = -1;
            int[] next = new int[256];
            next[0] = -1;
            int stringLength = s.Length;
            while(i < stringLength)
            {
                if (j == -1 || s[i] == s[j])
                {
                    i++;
                    j++;
                    if (s[i] != s[j])
                        next[i] = j;
                    //回溯后若与原位置相等，则再次回溯
                    else
                        i = next[j];
                }
                else
                {
                    j = next[j];
                }
            }
            return next;
        }
```



好了，KMP算法与它的改进我们算是介绍完了。

无疑，KMP算法与暴力查找算法相比，当匹配不成功时，i 指针不变，利用next数组对j进行回溯，从而避免了不必要的指针回溯，提高了查找效率。

但KMP算法并不是最快的字符串查找算法。

1977年，德克萨斯大学的Robert S. Boyer教授和J Strother Moore教授发明了一种新的字符串匹配算法：Boyer-Moore算法，简称BM算法

1990年，Daniel M.Sunday提出Sunday算法。

这两种算法比KMP速度更快。

大家有兴趣可以研究一下。



**END**





