---
title: 后缀自动机(SAM)的实现与应用
date: 2019-08-16 16:50:08
tags:
  - SAM
categories:	
  - Algorithm
  - ACM Solution
---

## 原理

后缀自动机(Suffix Automaton,简称SAM)是一个能解决许多字符串相关问题的有力的数据结构。举几个例子

* 一个字符串是否出现在另一个字符串中
* 一个字符串包含的各不相同的子串数目
* 查询一个字符串在另一个字符串中的出现次数

* 一个字符串在另一个字符串中第一次出现位置或者所有出现位置
* 求两个或多个字符串的最长公共子串

上面这几个例子在构建好SAM之后都可以在线性时间内解决

具体的原理可以参考这篇文章[^1],里面不仅给出了算法流程还给出了算法的正确性证明.

[^1]: [https://oi-wiki.org/string/sam/](https://oi-wiki.org/string/sam/)

## 实现

首先我们假设字符集大小为常数。如果字符集大小不是常数，SAM 的时间复杂度就不是线性的。从一个结点出发的转移存储在支持快速查询和插入的平衡树(map)中。因此如果我们记$\sum$为字符集，$|\sum|$为字符集大小，则算法的渐进时间复杂度为$O(nlog|\sum|)$，空间复杂度为$O(n)$。然而如果字符集足够小，可以不写平衡树，以空间换时间将每个结点的转移存储为长度为$|\sum|$的数组（用于快速查询）和链表（用于快速遍历所有可用关键字）。这样算法的时间复杂度为$O(n)$，空间复杂度为$O(n|\sum|)$。

这两种转移方式我都实现了一遍,并且在SAM基础上解决了前面列举的问题,下面是具体实现的代码

### 平衡树(map)版本

<details>
<summary>展开代码</summary>
```c++
#include <bits/stdc++.h>
using namespace std;

// 后缀自动机，可以在O(n)复杂度内创建一个字符串的SAM
// 最多2*n - 1个状态节点(n >= 2)，最多3*n - 4个转换(n >= 3)
namespace SAM {
​    struct state {
​        // SAM必须变量
​        int len, link;  // len表示状态包含的后缀字符串中的最长串长度,link为后缀链接len(link[v]) + 1 = minlen(v)
​        std::map<char, int> next;

        // 用于求子串出现次数的变量
        int cnt; // 子串出现次数
    
        // firstpos用于求子串第一次出现位置,配合is_clone和inv_link可用于求子串所有出现位置
        int firstpos;  // 子串第一次出现位置
        bool is_clone;  // 标志节点是否是复制的
        vector<int> inv_link; // 节点的后缀引用列表,该节点对应串是列表内节点的后缀
    
        // ml和nl用于求多字符串的最长公共子串
        int nl;  // 当前字符串与状态节点对应最长后缀的公共子串长
        int ml;  // 所有字符串与状态节点对应最长后缀的公共子串长中的最小长度
    }; // 状态节点，一个状态对应一个endpoint集合
    
    const int MAXLEN = 100000;  // 字符串长度限制
    state st[MAXLEN * 2];
    int sz, last;
    
    // 初始化
    void sam_init() {
        for (int i = 0; i < sz; i++) {
            st[i].next.clear();
            st[i].inv_link.clear();
        }
        st[0].len = 0;
        st[0].link = -1;
        sz = 1;
        last = 0;
    }
    
    // 向sam中插入字符
    void sam_extend(char c) {
        int cur = sz++;
        st[cur].len = st[last].len + 1;
        st[cur].ml = st[cur].len;
        st[cur].nl = 0;
        st[cur].firstpos = st[cur].len - 1;
        st[cur].is_clone = false;
        st[cur].cnt = 1;
        int p = last;
        while (p != -1 && !st[p].next.count(c)) {
            st[p].next[c] = cur;
            p = st[p].link;
        }
        if (p == -1) {
            st[cur].link = 0;
        } else {
            int q = st[p].next[c];
            if (st[p].len + 1 == st[q].len) {
                st[cur].link = q;
            } else {
                int clone = sz++;
                st[clone].len = st[p].len + 1;
                st[clone].ml = st[clone].len;
                st[clone].nl = 0;
                st[clone].is_clone = true;
                st[clone].firstpos = st[q].firstpos;
                st[clone].cnt = 0;
                st[clone].next = st[q].next;
                st[clone].link = st[q].link;
                while (p != -1 && st[p].next.count(c) && st[p].next[c] == q) {
                    st[p].next[c] = clone;
                    p = st[p].link;
                }
                st[q].link = st[cur].link = clone;
            }
        }
        last = cur;
    }
}

// 获取子串数目
bool vis[SAM::MAXLEN * 2];
int get_substr_num(int idx) {
​    int res = (idx == 0 ? 0 : SAM::st[idx].len - SAM::st[SAM::st[idx].link].len);
​    map<char, int>::iterator iter = SAM::st[idx].next.begin();
​    for (; iter != SAM::st[idx].next.end(); iter++) {
​        if (!vis[iter->second]) res += get_substr_num(iter->second);
​    }
​    vis[idx] = true;
​    return res;
}

// 子串查询，判断子串s是否包含在SAM对应的字符串中
bool lookup(char *s) {
​    int idx = 0;
​    for (int i = 0; s[i]; i++) {
​        if (SAM::st[idx].next.count(s[i])) 
​            idx = SAM::st[idx].next[s[i]]; 
​        else return false;
​    }
​    return true;
}

// 路径计数(可用于计算子串数目)
int d[SAM::MAXLEN * 2]; // d[i]表示从节点i出发的路径数目
int path_count(int idx) {
​    // 从根节点出发的路径数目即为子串数目
​    d[idx] = 0;
​    map<char, int>::iterator iter = SAM::st[idx].next.begin();
​    for (; iter != SAM::st[idx].next.end(); iter++) {
​        d[idx] += 1 + path_count(iter->second);
​    }
​    return d[idx];
}

// 查找字典序第k大的子串
bool find_kth_substr(int k, string &res, int idx) {
​    map<char, int>::iterator iter = SAM::st[idx].next.begin();
​    for (; iter != SAM::st[idx].next.end(); iter++) {
​        int t = 1 + d[iter->second];
​        if (k > t) k -= t;
​        else {
​            res += iter->first;
​            if (k > 1) find_kth_substr(k - 1, res, iter->second);
​            return true;
​        }
​    }
​    return false;
}

// 查询子串出现次数
int csort[SAM::MAXLEN + 1]; // 用于对len计数排序
int idsort[2 * SAM::MAXLEN];  // 存储按len排好序的索引id
int query_count(char *s, int idx) {
   std::map<char, int>::iterator iter = SAM::st[idx].next.begin();
   for (; iter != SAM::st[idx].next.end(); iter++) {
​       if (iter->first == s[0]) {
​           return !s[1] ? SAM::st[iter->second].cnt : query_count(s + 1, iter->second);
​       }
   }
   return 0;
}

// 查询子串第一次出现位置,返回对应节点索引号,不存在则返回-1
int query_firstpos(char *s, int idx) {
​    std::map<char, int>::iterator iter = SAM::st[idx].next.begin();
​    for (; iter != SAM::st[idx].next.end(); iter++) {
​        if (iter->first == s[0]) {
​            return !s[1] ? iter->second : query_firstpos(s + 1, iter->second);
​        }
​    }
​    return -1;
}

// 查询子串所有出现位置
void query_allpos(int v, int plen, vector<int> &res) {
​    if (!SAM::st[v].is_clone) res.push_back(SAM::st[v].firstpos - plen + 1);
​    for (size_t i = 0; i < SAM::st[v].inv_link.size(); i++) query_allpos(SAM::st[v].inv_link[i], plen, res);
} 

// 最长公共子串
string LCS(string t) {
​    int v = 0, l = 0, best = 0, bestpos = 0;
​    for (size_t i = 0; i < t.size(); i++) {
​        while (v && !SAM::st[v].next.count(t[i])) {
​            v = SAM::st[v].link;
​            l = SAM::st[v].len;
​        }
​        if (SAM::st[v].next.count(t[i])) {
​            v = SAM::st[v].next[t[i]];
​            l++;
​        }
​        if (l > best) {
​            best = l;
​            bestpos = i;
​        }
​    }
​    return t.substr(bestpos - best + 1, best);
}

// 多个字符串的最长公共前缀
// 依据SAM的状态节点id在SAM中提取指定长度的子串,存在则返回真
// 并将结果写到lcs中否则返回false
bool get_substr(int idx, int sid, size_t len, string &lcs) {
​    if (idx != 0 && idx == sid) return true; 
​    map<char, int>::iterator iter = SAM::st[idx].next.begin();
​    for (; iter != SAM::st[idx].next.end(); iter++) {
​        if (get_substr(iter->second, sid, len, lcs)) {
​            lcs += iter->first;
​            if (idx != 0) return true;
​            else {
​                if (lcs.size() == len) {
​                    reverse(lcs.begin(), lcs.end());
​                    return true;
​                } else {
​                    lcs.clear();    
​                }
​            }
​        }
​    }
​    return false;
}

// 时间复杂度为各字符串长总和
void LCS_Mul(int n, string &lcs) {
​    int ans = 0, nid = 0;
​    string s;
​    while (n--) {
​        cin >> s;
​        int tnl = 0, p = 0;
​        for (size_t j = 0; j < s.size(); j++) {
​            if (SAM::st[p].next.count(s[j])) {
​                tnl++;
​                p = SAM::st[p].next[s[j]];
​            } else {
​                while (p && !SAM::st[p].next.count(s[j])) p = SAM::st[p].link;
​                if (SAM::st[p].next.count(s[j])) {
​                    tnl = SAM::st[p].len + 1;
​                    p = SAM::st[p].next[s[j]];
​                } else {
​                    tnl = 0;
​                }
​            }
​            SAM::st[p].nl = max(SAM::st[p].nl, tnl);
​        }
​        for (int j = SAM::sz - 1; j > 0; j--) {
​            p = idsort[j];
​            if (SAM::st[p].nl < SAM::st[p].ml) SAM::st[p].ml = SAM::st[p].nl;
​            if (SAM::st[p].link && SAM::st[SAM::st[p].link].nl < SAM::st[p].nl)
​                SAM::st[SAM::st[p].link].nl = SAM::st[p].nl;
​            SAM::st[p].nl = 0;
​        }
​    }
​    for (int i = 1; i < SAM::sz; i++) {
​        if (SAM::st[i].ml > ans) {
​            ans = SAM::st[i].ml;
​            nid = i;
​        }
​    }
​    cout << ans << endl;
​    get_substr(0, nid, ans, lcs);
}


char instr[SAM::MAXLEN];
int main() {
​    cin >> instr;
​    SAM::sam_init();
​    for (int i = 0; instr[i]; i++) SAM::sam_extend(instr[i]);
​    cout << "SAM节点数: " << SAM::sz << endl;  // SAM节点数

    // 获取子串总数
    memset(vis, false, sizeof(vis));
    cout << get_substr_num(0) << " " << path_count(0) << endl;
    
    // 查询串s是否在母串中
    char s[100];
    cout << "查询子串: ";
    cin >> s;
    if (lookup(s)) cout << "exists!" << endl;
    else cout << "not exists!" << endl;
    
    // 查找字典序第k大的子串
    string substr_k = "";
    cout << "查询第k大子串:";
    int k;
    cin >> k;
    if (find_kth_substr(k, substr_k, 0))
        cout << "第" << k << "大子串: " << substr_k << endl;
    else
        cout << "不存在," << k << "已超过" << instr << "子串总数" << endl;
    
    // 查询子串出现次数
    cout << "查询子串出现次数: ";
    cin >> s;
    // 对len计数排序
    int m = 0;
    memset(csort, 0, sizeof(csort));
    for (int i = 1; i < SAM::sz; i++) {
        csort[SAM::st[i].len]++;
        m = max(m, SAM::st[i].len);
    }
    for (int i = 2; i <= m; i++) csort[i] += csort[i - 1];
    for (int i = 1; i < SAM::sz; i++) idsort[csort[SAM::st[i].len]--] = i;
    for (int i = SAM::sz - 1; i > 0; i--) {
        SAM::st[SAM::st[idsort[i]].link].cnt += SAM::st[idsort[i]].cnt;
    }
    cout << "子串数目: " << query_count(s, 0) << endl;
    
    // 查询子串第一次出现位置,不存在则返回-1,否则返回节点索引对应到第一次出现结束位置
    cout << "查询子串第一次出现位置: ";
    cin >> s;
    int fid = query_firstpos(s, 0);
    if (fid == -1)
        cout << "子串不存在!" << endl;
    else
        cout << SAM::st[fid].firstpos - int(strlen(s)) + 1 << endl;
    
    // 查询子串的所有出现位置(依赖于子串第一次出现节点)
    cout << "查询子串所有出现位置: ";
    cin >> s;
    fid = query_firstpos(s, 0);
    if (fid == -1) {
        cout << "子串不存在!" << endl;
    } else {
        for (int i = 1; i < SAM::sz; i++) SAM::st[SAM::st[i].link].inv_link.push_back(i);
        vector<int> occur_pos; // 所有出现的起始位置存放在occur_pos中
        query_allpos(fid, strlen(s), occur_pos);
        for (size_t i = 0; i < occur_pos.size(); i++) cout << occur_pos[i] << " ";
        cout << endl;
    }
    
    // 两个字符串的最长公共子串
    cout << "两个字符串的最长公共子串: ";
    string s1;
    cin >> s1;
    cout << LCS(s1) << endl;
    
    // 多个字符串的最长公共子串
    cout << "多个字符串的最长公共子串: ";
    int n;
    cin >> n;
    // 对len计数排序
    int lens = 0; // len类别数
    memset(csort, 0, sizeof(csort));
    for (int i = 1; i < SAM::sz; i++) {
        csort[SAM::st[i].len]++;
        lens = max(lens, SAM::st[i].len);
    }
    for (int i = 2; i <= lens; i++) csort[i] += csort[i - 1];
    for (int i = 1; i < SAM::sz; i++) idsort[csort[SAM::st[i].len]--] = i;
    string lcs;
    LCS_Mul(n, lcs);
    cout << "多个子串的最长公共子串为" << lcs << ", 长度" << lcs.size() << endl;
    return 0;
}

```
</details>


### 数组(array)版本

<details>
<summary>展开代码</summary>
```c++
#include <bits/stdc++.h>
using namespace std;

// 后缀自动机，可以在O(n)复杂度内创建一个字符串的SAM
// 最多2*n - 1个状态节点(n >= 2)，最多3*n - 4个转换(n >= 3)
namespace SAM {
    struct state {
        // SAM必须变量
        int len, link;  // len表示状态包含的后缀字符串中的最长串长度,link为后缀链接len(link[v]) + 1 = minlen(v)
        int trans[26];

        // 用于求子串出现次数的变量
        int cnt; // 子串出现次数

        // firstpos用于求子串第一次出现位置,配合is_clone和inv_link可用于求子串所有出现位置
        int firstpos;  // 子串第一次出现位置
        bool is_clone;  // 标志节点是否是复制的
        vector<int> inv_link; // 节点的后缀引用列表,该节点对应串是列表内节点的后缀

        // ml和nl用于求多字符串的最长公共子串
        int nl;  // 当前字符串与状态节点对应最长后缀的公共子串长
        int ml;  // 所有字符串与状态节点对应最长后缀的公共子串长中的最小长度
    }; // 状态节点，一个状态对应一个endpoint集合

    const int MAXLEN = 100000;  // 字符串长度限制
    state st[MAXLEN * 2];
    char alpha[26]; // 小写字母表,限制输入字符串仅含小写字母
    int sz, last;

    // 初始化
    void sam_init() {
        for (int i = 0; i < sz; i++) {
            memset(st[i].trans, 0 , sizeof(st[i].trans));
            st[i].inv_link.clear();
        }
        for (int i = 0; i < 26; i++) {
            alpha[i] = i + 'a';
        }
        st[0].len = 0;
        st[0].link = -1;
        sz = 1;
        last = 0;
    }

    // 向sam中插入字符
    void sam_extend(char ch) {
        int c = ch - 'a';
        int cur = sz++;
        st[cur].len = st[last].len + 1;
        st[cur].ml = st[cur].len;
        st[cur].nl = 0;
        st[cur].firstpos = st[cur].len - 1;
        st[cur].is_clone = false;
        st[cur].cnt = 1;
        int p = last;
        while (p != -1 && !st[p].trans[c]) {
            st[p].trans[c] = cur;
            p = st[p].link;
        }
        if (p == -1) {
            st[cur].link = 0;
        } else {
            int q = st[p].trans[c];
            if (st[p].len + 1 == st[q].len) {
                st[cur].link = q;
            } else {
                int clone = sz++;
                st[clone].len = st[p].len + 1;
                st[clone].ml = st[clone].len;
                st[clone].nl = 0;
                st[clone].is_clone = true;
                st[clone].firstpos = st[q].firstpos;
                st[clone].cnt = 0;
                memcpy(st[clone].trans, st[q].trans, sizeof(st[q].trans));
                st[clone].link = st[q].link;
                while (p != -1 && st[p].trans[c] == q) {
                    st[p].trans[c] = clone;
                    p = st[p].link;
                }
                st[q].link = st[cur].link = clone;
            }
        }
        last = cur;
    }
}

// 获取子串数目
bool vis[SAM::MAXLEN * 2];
int get_substr_num(int idx) {
    int res = (idx == 0 ? 0 : SAM::st[idx].len - SAM::st[SAM::st[idx].link].len);
    for (int i = 0; i < 26; i++) {
        if (!SAM::st[idx].trans[i]) continue;
        if (!vis[SAM::st[idx].trans[i]]) res += get_substr_num(SAM::st[idx].trans[i]);
    }
    vis[idx] = true;
    return res;
}

// 子串查询，判断子串s是否包含在SAM对应的字符串中
bool lookup(char *s) {
    int idx = 0;
    for (int i = 0; s[i]; i++) {
        if (SAM::st[idx].trans[s[i] - 'a']) 
            idx = SAM::st[idx].trans[s[i] - 'a']; 
        else return false;
    }
    return true;
}

// 路径计数(可用于计算子串数目)
int d[SAM::MAXLEN * 2]; // d[i]表示从节点i出发的路径数目
int path_count(int idx) {
    // 从根节点出发的路径数目即为子串数目
    d[idx] = 0;
    for (int i = 0; i < 26; i++) {
        if (SAM::st[idx].trans[i])
            d[idx] += 1 + path_count(SAM::st[idx].trans[i]);
    }
    return d[idx];
}

// 查找字典序第k大的子串
bool find_kth_substr(int k, string &res, int idx) {
    for (int i = 0; i < 26; i++) {
        if (!SAM::st[idx].trans[i]) continue;
        int t = 1 + d[SAM::st[idx].trans[i]];
        if (k > t) k -= t;
        else {
            res += SAM::alpha[i];
            if (k > 1) find_kth_substr(k - 1, res, SAM::st[idx].trans[i]);
            return true;
        }
    }
    return false;
}

// 查询子串出现次数
int csort[SAM::MAXLEN + 1]; // 用于对len计数排序
int idsort[2 * SAM::MAXLEN];  // 存储按len排好序的索引id
int query_count(char *s, int idx) {
   for (int i = 0; i < 26; i++) {
       if (!SAM::st[idx].trans[i]) continue;
       if (SAM::alpha[i] == s[0]) {
           return !s[1] ? SAM::st[SAM::st[idx].trans[i]].cnt : query_count(s + 1, SAM::st[idx].trans[i]);
       }
   }
   return 0;
}

// 查询子串第一次出现位置,返回对应节点索引号,不存在则返回-1
int query_firstpos(char *s, int idx) {
    for (int i = 0; i < 26; i++) {
        if (!SAM::st[idx].trans[i]) continue;
        if (SAM::alpha[i] == s[0]) {
            return !s[1] ? SAM::st[idx].trans[i] : query_firstpos(s + 1, SAM::st[idx].trans[i]);
        }
    }
    return -1;
}

// 查询子串所有出现位置
void query_allpos(int v, int plen, vector<int> &res) {
    if (!SAM::st[v].is_clone) res.push_back(SAM::st[v].firstpos - plen + 1);
    for (size_t i = 0; i < SAM::st[v].inv_link.size(); i++) query_allpos(SAM::st[v].inv_link[i], plen, res);
} 

// 最长公共子串
string LCS(string t) {
    int v = 0, l = 0, best = 0, bestpos = 0;
    for (size_t i = 0; i < t.size(); i++) {
        while (v && !SAM::st[v].trans[t[i] - 'a']) {
            v = SAM::st[v].link;
            l = SAM::st[v].len;
        }
        if (SAM::st[v].trans[t[i] - 'a']) {
            v = SAM::st[v].trans[t[i] - 'a'];
            l++;
        }
        if (l > best) {
            best = l;
            bestpos = i;
        }
    }
    return t.substr(bestpos - best + 1, best);
}

// 多个字符串的最长公共前缀
// 依据SAM的状态节点id在SAM中提取指定长度的子串,存在则返回真
// 并将结果写到lcs中否则返回false
bool get_substr(int idx, int sid, size_t len, string &lcs) {
    if (idx != 0 && idx == sid) return true; 
    for (int i = 0; i < 26; i++) {
        if (!SAM::st[idx].trans[i]) continue;
        if (get_substr(SAM::st[idx].trans[i], sid, len, lcs)) {
            lcs += SAM::alpha[i];
            if (idx != 0) return true;
            else {
                if (lcs.size() == len) {
                    reverse(lcs.begin(), lcs.end());
                    return true;
                } else {
                    lcs.clear();    
                }
            }
        }
    }
    return false;
}

// 时间复杂度为各字符串长总和
void LCS_Mul(int n, string &lcs) {
    int ans = 0, nid = 0;
    string s;
    while (n--) {
        cin >> s;
        int tnl = 0, p = 0;
        for (size_t j = 0; j < s.size(); j++) {
            if (SAM::st[p].trans[s[j] - 'a']) {
                tnl++;
                p = SAM::st[p].trans[s[j] - 'a'];
            } else {
                while (p && !SAM::st[p].trans[s[j] - 'a']) p = SAM::st[p].link;
                if (SAM::st[p].trans[s[j] - 'a']) {
                    tnl = SAM::st[p].len + 1;
                    p = SAM::st[p].trans[s[j] - 'a'];
                } else {
                    tnl = 0;
                }
            }
            SAM::st[p].nl = max(SAM::st[p].nl, tnl);
        }
        for (int j = SAM::sz - 1; j > 0; j--) {
            p = idsort[j];
            if (SAM::st[p].nl < SAM::st[p].ml) SAM::st[p].ml = SAM::st[p].nl;
            if (SAM::st[p].link && SAM::st[SAM::st[p].link].nl < SAM::st[p].nl)
                SAM::st[SAM::st[p].link].nl = SAM::st[p].nl;
            SAM::st[p].nl = 0;
        }
    }
    for (int i = 1; i < SAM::sz; i++) {
        if (SAM::st[i].ml > ans) {
            ans = SAM::st[i].ml;
            nid = i;
        }
    }
    cout << ans << endl;
    get_substr(0, nid, ans, lcs);
}


char instr[SAM::MAXLEN];
int main() {
    cin >> instr;  // 仅限小写字母
    SAM::sam_init();
    for (int i = 0; instr[i]; i++) SAM::sam_extend(instr[i]);
    cout << "SAM节点数: " << SAM::sz << endl;  // SAM节点数

    // 获取子串总数
    memset(vis, false, sizeof(vis));
    cout << get_substr_num(0) << " " << path_count(0) << endl;

    // 查询串s是否在母串中
    char s[100];
    cout << "查询子串: ";
    cin >> s;
    if (lookup(s)) cout << "exists!" << endl;
    else cout << "not exists!" << endl;

    // 查找字典序第k大的子串
    string substr_k = "";
    cout << "查询第k大子串:";
    int k;
    cin >> k;
    if (find_kth_substr(k, substr_k, 0))
        cout << "第" << k << "大子串: " << substr_k << endl;
    else
        cout << "不存在," << k << "已超过" << instr << "子串总数" << endl;

    // 查询子串出现次数
    cout << "查询子串出现次数: ";
    cin >> s;
    // 对len计数排序
    int m = 0;
    memset(csort, 0, sizeof(csort));
    for (int i = 1; i < SAM::sz; i++) {
        csort[SAM::st[i].len]++;
        m = max(m, SAM::st[i].len);
    }
    for (int i = 2; i <= m; i++) csort[i] += csort[i - 1];
    for (int i = 1; i < SAM::sz; i++) idsort[csort[SAM::st[i].len]--] = i;
    for (int i = SAM::sz - 1; i > 0; i--) {
        SAM::st[SAM::st[idsort[i]].link].cnt += SAM::st[idsort[i]].cnt;
    }
    cout << "子串数目: " << query_count(s, 0) << endl;

    // 查询子串第一次出现位置,不存在则返回-1,否则返回节点索引对应到第一次出现结束位置
    cout << "查询子串第一次出现位置: ";
    cin >> s;
    int fid = query_firstpos(s, 0);
    if (fid == -1)
        cout << "子串不存在!" << endl;
    else
        cout << SAM::st[fid].firstpos - int(strlen(s)) + 1 << endl;

    // 查询子串的所有出现位置(依赖于子串第一次出现节点)
    cout << "查询子串所有出现位置: ";
    cin >> s;
    fid = query_firstpos(s, 0);
    if (fid == -1) {
        cout << "子串不存在!" << endl;
    } else {
        for (int i = 1; i < SAM::sz; i++) SAM::st[SAM::st[i].link].inv_link.push_back(i);
        vector<int> occur_pos; // 所有出现的起始位置存放在occur_pos中
        query_allpos(fid, strlen(s), occur_pos);
        for (size_t i = 0; i < occur_pos.size(); i++) cout << occur_pos[i] << " ";
        cout << endl;
    }

    // 两个字符串的最长公共子串
    cout << "两个字符串的最长公共子串: ";
    string s1;
    cin >> s1;
    cout << LCS(s1) << endl;

    // 多个字符串的最长公共子串
    cout << "多个字符串的最长公共子串: ";
    int n;
    cin >> n;
    // 对len计数排序
    int lens = 0; // len类别数
    memset(csort, 0, sizeof(csort));
    for (int i = 1; i < SAM::sz; i++) {
        csort[SAM::st[i].len]++;
        lens = max(lens, SAM::st[i].len);
    }
    for (int i = 2; i <= lens; i++) csort[i] += csort[i - 1];
    for (int i = 1; i < SAM::sz; i++) idsort[csort[SAM::st[i].len]--] = i;
    string lcs;
    LCS_Mul(n, lcs);
    cout << "多个子串的最长公共子串为" << lcs << ", 长度" << lcs.size() << endl;
    return 0;
}

```
</details>

