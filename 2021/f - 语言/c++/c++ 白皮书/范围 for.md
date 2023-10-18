范围 `for` 是用来顺序遍历一个序列中所有元素的语句。例如：

```cpp
void use(vector<int>& v, list<string>& lst)
{
    for (int x : v) cout << x << '\n';
    int sum = 0;
    for (auto i : {1,2,3,5,8}) sum+=i; // 初始化列表是一个序列
    for (string& s : lst) s += ".cpp"; // 使用引用允许遍历时修改
}
```

它最初是由 Thorsten Ottosen（丹麦奥尔堡大学）提出的，理由是“基本上任何现代编程语言都内置了某种形式的 for each” [Ottosen 2005]。我通常不认为“别人都有了”是个好的论据，但在这一情况下，真正的要点是，简单的范围循环可以简化一种最常见的操作，并提供了优化的机会。所以，范围 `for` 完美符合我对 C++ 的总体设计目标。它直接表达应该做什么，而不是详细描述如何做。它的语法简洁，语义明晰。

由于更简单和更明确，范围 `for` 语句消除了一些“微不足道”然而常见的错误：

```cpp
void use(vector<int>& v, list<string>& lst)
{
    for (int i=0; i<imax; ++i)
        for (int j=0; i<imax; ++j) ...  // 错误的嵌套循环

    for (int i=0; i<=max; ++i) ...      // 多循环了一次的错误
}
```

尽管范围 `for` 够简单了，它在这些年还是有些变化。Doug Gregor 曾建议使用 C++0x 中的概念来修改范围 `for`，方案优雅并且得到了批准 [Ottosen et al. 2007]。我还记得他在我在得州的办公室里写这个提案的场景，但很遗憾，后来因为删除了 C++0x 的概念，我们不得不回退了那些修改。在 2016 年，它还做过一点小修改，以配合 Ranges TS所支持的无限序列。