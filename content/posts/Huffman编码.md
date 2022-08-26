---
title: "Huffman编码"
subtitle: ""
date: 2022-08-25T17:26:53+02:00
draft: false
author: "spike"
authorLink: ""
authorEmail: ""
description: ""
keywords: ""
license: ""
comment: false
weight: 0

tags:
- algorithms
- c#
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



### Huffman编码

> **霍夫曼编码**（英语：Huffman Coding），又译为**哈夫曼编码**、**赫夫曼编码**，是一种用于[无损数据压缩](https://zh.wikipedia.org/wiki/无损数据压缩)的[熵编码](https://zh.wikipedia.org/wiki/熵编码)（权编码）[算法](https://zh.wikipedia.org/wiki/演算法)。由美国计算机科学家[大卫·霍夫曼](https://zh.wikipedia.org/wiki/大衛·霍夫曼)（David Albert Huffman）在1952年发明，是一种变长的编码方式，对出现频率高的字符采用较短的编码，而对出现频率较低的字符采用较长的编码（在对字符进行变长编码时，注意要满足**前缀码规则：任意字符的编码不能是其他字符编码的前缀**）

<!--more-->

要得到Huffman编码，首先需要构建Huffman树。我们首先来介绍一下Huffman树

#### Huffman树

在我们构造Huffman树之前，我们需要声明几个定义：

1. **路径**：连接两个结点的分支，构成这两个结点的路径
2. **路径长度**：路径的分支数
3. **结点的权**：为结点赋予的一个有意义的实数
4. **结点的带权路径长度**：根结点到该节点的 路径长度 乘以 该结点 的权值，及该结点的带权路径长度
5. **树的带权路径长度**：树中  所有叶子结点的带权路径长度 之和
6. **Huffman树**：$n$个带权叶子 构成的二叉树中，带权路径WPL最小的二叉树称为 Huffman树

##### Huffman树的构造步骤

1. 根据给定的$n$个权值$w_1,w_2,w_3,...,w_n$，构成 含$n$个二叉树 的森林$F={T_1,T_2,...,T_n}$，其中每个二叉树仅含一个权值为$w_i$的根结点，且无左右子树（即构成n个只有一个结点的二叉树）
2. 从森林中选取两个权值最小的 二叉树作为新树的左右子树列来构成一个新树（注意，**权值相对较小的为左子树**），新树的根结点权值即左右子树权值之和
3. 从森林$F$中删除构建新树的两棵子树，同时将新树加入到森林$F$中
4. 重复步骤2、3，直到森林$F$中只有一棵树，即得到Huffman树

举个栗子：

给定权集：{2,3,4,7,8,9}，构造一棵Huffman树：

<img src="../../../../../img/image-20220825172333175.png" alt="image-20220825172333175" style="zoom:50%;" />

构造过程如上所示。

##### Huffman编码的应用

我们介绍了Huffman树是如何来构造的，那么Huffman编码是如何应用到实际工作中的呢？

我们给定一个字符串，统计每个字符出现的频数，作为我们的权集，来构造一棵Huffman树，再通过我们构造的Huffman树来得到每个字符的编码，进而得到字符串的编码。

举个栗子，我们给定字符串"state,seat,act,tea,cat,set,a,eat"

通过Huffman树得到Huffman编码：

```
a:00
,:01
t:10
e:110
c:1110
s:1111
(整个字符串的编码)1111100010110011111110001001001110100110110000111100010011111110100100011100010
```



##### Huffman编码

那么接下来我们来一探究竟，看看我们是如何通过一棵Huffman树来得到每个叶子结点的Huffman编码呢？

我们需要对我们构造的Huffman树做一个小小的操作：在所有的左分支上标$0$，所有的右分支上标$1$。（即在每个结点到左孩子的边上写上数字$0$，到右孩子的边上写上数字$1$）

对于上面Huffman树，我们就得到：

<img src="../../../../../img/image-20220825172432315.png" alt="image-20220825172432315" style="zoom:50%;" />



通过这样的处理我们就能得到每个字符的编码了。

每个字符的编码即 **根结点到与该字符对应的叶子结点 的路径上所有编码的连接体**

对于上图中权值为$2$的叶子结点，其编码就为 $0100$



#### Huffman编码的C#代码实现

介绍完了Huffman编码的基本原理，我们来给出他的代码实现:

- 首先给出二叉树树结点的实现代码：

```C#
/// <summary>
/// 定义二叉树结点
/// </summary>
/// <typeparam name="T">二叉树结点数据域的数据类型</typeparam>
public class BinTreeNode<T> : IComparable<BinTreeNode<T>> where T : IComparable<T>
{
        /// <summary>
        /// 结点数据域所含数据
        /// </summary>
        private T _data;

        /// <summary>
        /// 获取或设置结点数据
        /// </summary>
        public T Data
        {
            get
            {
                return _data;
            }
            set
            {
                if (value == null)
                    throw new ArgumentNullException();
                _data = value;
            }
        }
        /// <summary>
        /// 获取或设置结点的左子树
        /// </summary>
        public BinTreeNode<T> LeftChild { get; set; }
        /// <summary>
        /// 获取或设置结点的右子树
        /// </summary>
        public BinTreeNode<T> RightChild { get; set; }

        /// <summary>
        /// 构造函数，初始化结点
        /// </summary>
        /// <param name="data">初始化结点的数据</param>
        public BinTreeNode(T data)
        {
            LeftChild = null;
            _data = data;
            RightChild = null;
        }
        /// <summary>
        /// 构造函数，初始化结点
        /// </summary>
        /// <param name="lChild">初始化结点的左子树</param>
        /// <param name="data">初始化结点的数据</param>
        /// <param name="rChild">初始化结点的右子树</param>
        public BinTreeNode(BinTreeNode<T> lChild, T data, BinTreeNode<T> rChild)
        {
            LeftChild = lChild;
            _data = data;
            RightChild = rChild;
        }
        public int CompareTo(BinTreeNode<T> other)
        {
            if (other == null)
                throw new ArgumentNullException();
            return _data.CompareTo(other.Data);
        }
}
```

- 接着给出Huffman树结点的实现代码（与二叉树结点相比，每个结点多了一个权重Weight属性）

```C#
/// <summary>
/// Huffman树结点定义(base()向父类构造函数 传递参数)
/// </summary>
public class HuffManTreeNode : BinTreeNode<char> //继承了二叉树结点
{
    /// <summary>
    /// Huffman树结点的权重
    /// </summary>
    public int Weight { get; set; }

    /// <summary>
    /// 构造Huffman树结点
    /// </summary>
    /// <param name="data"></param>
    /// <param name="weight"></param>
    public HuffManTreeNode(char data, int weight) : base(data)
    {            
    Weight = weight;
    }
    /// <summary>
    /// 构造 Huffman树结点 所含数据为空
    /// </summary>
    /// <param name="weight"></param>
    public HuffManTreeNode(int weight) : base('\0')
    {
    Weight = weight;
    }
}
```

- 接着给出Huffman编码字典的实现代码，用以存储字符及字符对应的Huffman编码

```C#
/// <summary>
/// 构造一个数据类型，存储字符及字符的huffman编码
/// </summary>
public class HuffmanDicItem
{
    /// <summary>
    /// 字符
    /// </summary>
    public char Character { get; set; }
    /// <summary>
    /// 字符的huffman编码
    /// </summary>
    public string Code { get; set; }

    public HuffmanDicItem(char character, string code)
    {
    if (character == '\0')
    throw new Exception("字符为空");
    if (code == string.Empty)
    throw new Exception("编码为空");
    Character = character;
    Code = code;
    }
}
```

- 最后给出Huffman树的的实现代码（实现Huffman树的构造、Huffman编码、对Huffman编码进行解码）

```C#
public class HuffManTree
{
    /// <summary>
    /// 对字符串中的字符进行频数统计，构造森林
    /// </summary>
    /// <param name="str">待编码的字符串</param>
    /// <returns></returns>
    private List<HuffManTreeNode> CreateInitForest(string str)
    {
        if (string.IsNullOrEmpty(str))
            throw new ArgumentNullException();

        List<HuffManTreeNode> result = new List<HuffManTreeNode>(); //链表存储huffman树结点
        char[] charArray = str.ToCharArray();
        List<IGrouping<char, char>> lst = charArray.GroupBy(a => a).ToList(); // lambda表达式
        // ToList()?
        foreach(IGrouping<char, char> g in lst)
        {
            char data = g.Key;
            int weight = g.ToList().Count;
            HuffManTreeNode node = new HuffManTreeNode(data, weight);
            result.Add(node);
        }
        return result;
    }
    /// <summary>
    /// 构造Huffman树
    /// </summary>
    /// <param name="sources"></param>
    /// <returns></returns>
    private HuffManTreeNode CreateHuffmanTree(List<HuffManTreeNode> sources)
    {
        if (sources == null)
            throw new ArgumentNullException();
        if (sources.Count < 2)
            throw new Exception("少于两个结点，无法构成Huffman树");

        HuffManTreeNode root = default(HuffManTreeNode);
        bool isNext = true;

        while (isNext)
        {
            List<HuffManTreeNode> lst = sources.OrderBy(a => a.Weight).ToList();
            HuffManTreeNode first = lst[0];
            HuffManTreeNode second = lst[1];
            int Sum_weight = first.Weight + second.Weight;

            HuffManTreeNode node = new HuffManTreeNode(Sum_weight);
            node.LeftChild = first;
            node.RightChild = second;

            if (lst.Count == 2)
            {
                root = node;
                isNext = false;
            }
            else
            {
                sources = lst.GetRange(2, sources.Count - 2);
                sources.Add(node);
            }
        }
        return root;
    }
    /// <summary>
    /// 构造Huffman编码字典
    /// </summary>
    /// <param name="code"></param>
    /// <param name="current"></param>
    /// <returns></returns>
    private List<HuffmanDicItem> CreateHuffmanDict(string code, HuffManTreeNode current)
    {
        if (current == null)
            throw new ArgumentNullException();

        List<HuffmanDicItem> result = new List<HuffmanDicItem>();
        if (current.LeftChild == null && current.RightChild == null)
        {
            result.Add(new HuffmanDicItem(current.Data, code));
        }
        else
        {
            List<HuffmanDicItem> dictL = CreateHuffmanDict(code + "0", (HuffManTreeNode)current.LeftChild);
            List<HuffmanDicItem> dictR = CreateHuffmanDict(code + "1", (HuffManTreeNode)current.RightChild);

            result.AddRange(dictL);
            result.AddRange(dictR);
        }
        return result;
    }
    private List<HuffmanDicItem> CreateHuffmanDict(HuffManTreeNode root)
    {
        if (root == null)
            throw new ArgumentNullException();
        return CreateHuffmanDict(string.Empty, root);
    }
    /// <summary>
    /// 对字符串进行编码
    /// </summary>
    /// <param name="source"></param>
    /// <param name="lst"></param>
    /// <returns></returns>
    private string ToHuffmanCode(string source, List<HuffmanDicItem> lst)
    {
        if (string.IsNullOrEmpty(source))
            throw new ArgumentNullException();
        if (lst == null)
            throw new ArgumentNullException();

        string result = string.Empty;
        for (int i = 0; i < source.Length; i++)
        {
            result += lst.Single(a => a.Character == source[i]).Code;
        }
        return result;
    }
    public string StringToHuffmanCode(out List<HuffmanDicItem> dict, string str)
    {
        List<HuffManTreeNode> forest = CreateInitForest(str);
        HuffManTreeNode root = CreateHuffmanTree(forest);
        dict = CreateHuffmanDict(root);
        string result = ToHuffmanCode(str, dict);
        return result;
    }
    /// <summary>
    /// 对Huffman编码进行解码
    /// </summary>
    /// <param name="dict"></param>
    /// <param name="code"></param>
    /// <returns></returns>
    public string HuffmanCodeToString(List<HuffmanDicItem> dict, string code)
    {
        string result = string.Empty;
        for (int i = 0; i < code.Length;)
        {
            foreach(HuffmanDicItem Item in dict)
            {
                if (code[i] == Item.Code[0] && Item.Code.Length + i <= code.Length)
                {
                    string temp = code.Substring(i, Item.Code.Length);
                    if (temp == Item.Code)
                    {
                        result += Item.Character;
                        i += Item.Code.Length;
                        break;
                    }
                }
            }
        }
        return result;
    }
}
```

- 最后给出一个测试栗子，来检测我们的代码：

给定字符串“state,seat,act,tea,cat,set,a,eat”，对其进行Huffman编码

```C#
static void Main(string[] args)
{
    string str = "state,seat,act,tea,cat,set,a,eat";
    Console.WriteLine(str);

    HuffManTree huffmanTree = new HuffManTree();
    List<HuffmanDicItem> dic;
    string code = huffmanTree.StringToHuffmanCode(out dic, str); // 对给定字符串进行编码
    for (int i = 0; i < dic.Count; i++)
    {
    Console.WriteLine(dic[i].Character + ":" + dic[i].Code);

    }
    Console.WriteLine(code);

    string decode = huffmanTree.HuffmanCodeToString(dic, code); // 对Huffman编码进行解码
    Console.WriteLine(decode);
    Console.ReadLine();
}
```

运行结果如下：

![image-20220825172525386](../../../../../img/image-20220825172525386.png)



综上，我们便介绍完了Huffman树及Huffman编码，并给出了Huffman编码的实现代码。

可以看出Huffman编码与常规的定长编码相比，在避免多余的空间浪费，给出了一种更高效的变长编码方式
