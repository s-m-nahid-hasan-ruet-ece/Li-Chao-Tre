# **Before Start**
Recently, I have faced a problem in **Team Practice Contest** for **ACM ICPC** which was organized by **RUET Analytical Programming Lab**. I had no idea how to solve that problem in that time complexity during contest time. After the contest, one of my seniors told me that he solved that problem using **_Li Chao Tree_**.

# **Problem Discussion**
In that problem, there will be given an integer **_n (2≤n≤1,000,000)_** which was called the number of buildings. Next line an array of integers **_h (1≤hi≤1,000,000)_** will be given which is the heights of the buildings. The **_i-th_** building from left to right has width **_1_** and height **_hi_**. Among the **_n_** buildings, we have to find two buildings, say the **_i-th_** building and **_j-th_** building with **_i<j_**, such that **_(hi+hj)∗(j−i)_** is maximized.

The original problem statement is [here](https://codeforces.com/gym/102920/problem/L) .

# **Li Chao Tree**
Assume you're given a set of functions such that each two can intersect at most once. Let's keep in each vertex of a segment tree some function in such way, that if we go from root to the leaf it will be guaranteed that one of the functions we met on the path will be the one giving the minimum value in that leaf. Let's see how to construct it.

Assume we're in some vertex corresponding to half-segment **_[l,r)_** and the function fold is kept there and we add the function fnew. Then the intersection point will be either in **_[l;m)_** or in **_[m;r)_** where **_m=⌊l+r2⌋_**. We can efficiently find that out by comparing the values of the functions in points **_l_** and **_m_**. If the dominating function changes, then it is in **_[l;m)_** otherwise it is in **_[m;r)_**. Now for the half of the segment with no intersection we will pick the lower function and write it in the current vertex. You can see that it will always be the one which is lower in point **_m_**. After that we recursively go to the other half of the segment with the function which was the upper one. As you can see this will keep correctness on the first half of segment and in the other one correctness will be maintained during the recursive call. Thus we can add functions and check the minimum value in the point in **_O(log[Cε−1])_**.

Here is the illustration of what is going on in the vertex when we add new function:

![](https://raw.githubusercontent.com/e-maxx-eng/e-maxx-eng/master/img/li_chao_vertex.png)

Let's go to implementation now. Once again we will use complex numbers to keep linear functions.

# **Implementation**
We use line as an example here:

```cpp - C++
    #define ll long long int 
    const ll sz = 1000010;
    ll h[sz];
    bool exist[4*sz];

    struct Line
    {
       ll m,c;
       inline ll f(ll x) {return h[m]*x - m*h[x] +c;}
    } tree[4*sz];
```


On every node of the segment tree, we store the line that maximize(or minimize) the value of the middle i.e. if the interval of the node is **_[L,R)_** , then the line stored on it maximize(or minimize) **_(L+R)/2_**.


## **Insert**

Suppose that we are inserting a new line to the node which corresponding interval is **_[L,R)_**. For conveinence, we assume that the line on the node have a smaller slope. Let **_mid=(L+R)/2_**. Then, we have two cases to consider (red is the original line in the node):

1. **_red (mid) < blue (mid)_**

![](https://robert1003.github.io/assets/images/li-chao-segment-tree/smaller.png)


In this case, we should replace red with blue. But should we discard red? No! As you can see, somewhere in **_[L,mid)_** needs red. Therefore, we should pass this line to the node with interval **_[L,mid)_**, which is its left son.

**_2. red (mid) > blue (mid)_**

![](https://robert1003.github.io/assets/images/li-chao-segment-tree/greater.png)

Similarly, we should keep red in this node and pass blue to its right son, whose corresponding interval is **_[mid,R)_**.

```cpp - C++
    void insert(ll lo, ll hi, Line line, ll node)
    {
        exist[node] = 1;

        if(lo==hi)
        {
            if(line.f(lo) > tree[node].f(lo))
               tree[node]= line;
        }
        return;

        ll mid = lo+hi >> 1;
        bool left = line.f(lo) > tree[node].f(lo);
        bool m = line.f(mid) > tree[node].f(mid);
        
        if(m) swap(tree[node], line);
        if(left!=m) insert(lo,mid,line,node<<1);
        else insert(mid+1,hi,line,node<<1|1);
    }
```
The time complexity is **_O(logC)_**, where **_C_** is the size of range we maintain.

## **Query**
From the way we insert lines, we know that we only need to consider intervals that conatins the point we want to ask.

```cpp - C++
    ll query(ll lo, ll hi, ll idx, ll node)
    {
        if(lo== hi)
        {
            return tree[ndoe].f(idx);
        }

        ll mid = lo+hi >>1, ret = tree[node].f(idx);

        if(idx<= mid && exist[node<<1]) ret = max(ret, query(lo, mid, idx, node<<1));
        else if(idx > mid && exist[node<<1|1]) ret = max(ret, query(mid+1, hi, idx, node<<1|1));

        return ret;

    }
```
The time complexity is still **_O(logC)_**, as we will only pass **_O(logC)_** nodes.

# **More Li Chao Tree Problems**
* [Blue Mary's Company](https://www.lydsy.com/JudgeOnline/problem.php?id=1568)
* [Polynomials](https://www.codechef.com/NOV17/problems/POLY)
* [Escape Through Leaf](https://www.codeforces.com/problemset/problem/932/F)
* [Squared Ends](https://www.csacademy.com/contest/archive/task/squared-ends)
* [Game](https://www.lydsy.com/JudgeOnline/problem.php?id=4515)
* [Robot](https://www.lydsy.com/JudgeOnline/problem.php?id=3938)


# **References**
* [A Simple Introduction to Li-Chao Segment Tree](https://robert1003.github.io/2020/02/06/li-chao-segment-tree.html)
* [Convex hull trick and Li Chao tree](https://cp-algorithms.com/geometry/convex_hull_trick.html)
